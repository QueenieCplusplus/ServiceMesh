# ServiceMesh
Network Architecture of Migrating Microservices of App from On-premise to Cloud

using Istio Mesh to connect and monitor those microsevices in a secure way.

![](https://cloud.google.com/solutions/images/supporting-your-migration-with-istio-mesh-expansion-service-mesh.svg)

![](https://cloud.google.com/solutions/images/supporting-your-migration-with-istio-mesh-expansion-legacy-data-center.svg)

![gui](https://cloud.google.com/solutions/images/supporting-your-migration-with-istio-mesh-expansion-visualize-mesh.png)

# Core Steps:

(1) prepare istio.

(2) access the legacy (on-premise) enviroment.

(3) provision the target enviroment.

(4) config a service mesh.

(5) add service in (1) into (3).

(6) deploy services in (3).

(7) split traffic by setting up Routing Rule.

(8) route traffice to (3).

(9) retire (1).

(10) use Kiali to visualize service mesh

# Istio

from step 1

> initialize Istio

* 1.1, after setting up Region & Zone of GCE, then init the istio version env variable in sys.

      export ISTIO_VERSION=1.1.1

* 1.2, setup PATH for env.

      ISTIO_PATH="$HOME"/istio-"$ISTIO_VERSION"
      
      HELM_VERSION=v2.13.0
      
      HELM_PATH="$HOME"/helm-"$HELM_VERSION"

* 1.3, activate cloud shell, download and install istio, then check istion version.

      wget https://github.com/istio/istio/releases/download/"$ISTIO_VERSION"/istio-"$ISTIO_VERSION"-linux.tar.gz
      
      tar -xvzf istio-"$ISTIO_VERSION"-linux.tar.gz

* 1.4, download and install Helm CLI tool for istio.

       wget https://storage.googleapis.com/kubernetes-helm/helm-"$HELM_VERSION"-linux-amd64.tar.gz
       
       tar -xvzf helm-"$HELM_VERSION"-linux-amd64.tar.gz

       mv linux-amd64 "$HELM_PATH"

* 1.5, init the istio account and email address for the GCE that runs a specific microservice.

      GCE_SERVICE_ACCOUNT_NAME=istio-migration-gce
      // init env variable to sys
      
      gcloud iam service-accounts create "$GCE_SERVICE_ACCOUNT_NAME" 
         --display-name="$GCE_SERVICE_ACCOUNT_NAME"
      
      
      GCE_SERVICE_ACCOUNT_EMAIL="$(gcloud iam service-accounts list \
          --format='value(email)' \
          --filter=displayName:"$GCE_SERVICE_ACCOUNT_NAME")"

* 1.6, bind role for the specific microservice in app.

      gcloud projects add-iam-policy-binding "$(gcloud config get-value project 2> /dev/null)" \
                 --member serviceAccount:"$GCE_SERVICE_ACCOUNT_EMAIL" \
                 --role roles/compute.viewer
 
 * 1.7 create GCE VM for app in legacy env.
 
       export GCE_INSTANCE_NAME=legacy-vm
       // init env varibale in sys.
       
       gcloud compute instances create "$GCE_INSTANCE_NAME" \
          --boot-disk-device-name="$GCE_INSTANCE_NAME" \
          --boot-disk-size=10GB \
           --boot-disk-type=pd-ssd \
          --image-family=ubuntu-1804-lts \
          --image-project=ubuntu-os-cloud \
          --machine-type=n1-standard-1 \
          --metadata-from-file startup-script="$HOME"/solutions-istio-mesh-expansion-migration/gce-startup.sh \
          --scopes=storage-ro,logging-write,monitoring-write,service-control,service-management,trace \
          --service-account="$GCE_SERVICE_ACCOUNT_EMAIL" \
          --tags=bookinfo-legacy-vm
          
        // create GCE.
        
        [output]
        
        NAME           ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
       legacy-vm      us-east1-b  n1-standard-1               10.142.0.38  34.73.53.145  RUNNING
        
# WAF

from step 2 

> setup fire wall rule for both of the legacy and destination evn.

* 2.1, create a fire wall rule, and its options called desciption, action, rules.

        gcloud compute firewall-rules create bookinfo \
              --description="Bookinfo App rules" \
              --action=ALLOW \
              --rules=tcp:9080,tcp:9081,tcp:9082,tcp:9083,tcp:9084 \
              --target-tags=bookinfo-legacy-vm

