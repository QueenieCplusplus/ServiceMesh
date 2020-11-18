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

(4) config a service mesh by creating GKE cluster with node pool and install Istio in cluster.

(5) add service in (1) into (3).

(6) deploy services in (3).

(7) split traffic by setting up Routing Rule.

(8) route traffice to (3).

(9) retire (1).

(10) use Kiali to visualize service mesh

# Istio Account

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

# Legacy

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

# Target

from step 3

> provision the target env.

* 3.1, 

      GKE_SERVICE_ACCOUNT_NAME=istio-migration-gke

      gcloud iam service-accounts create "$GKE_SERVICE_ACCOUNT_NAME" \
          --display-name="$GKE_SERVICE_ACCOUNT_NAME
          
      GKE_SERVICE_ACCOUNT_EMAIL="$(gcloud iam service-accounts list \
         --format='value(email)' \
         --filter=displayName:"$GKE_SERVICE_ACCOUNT_NAME")"
         
* 3.2, grant role to the microservices.

      gcloud projects add-iam-policy-binding \
          "$(gcloud config get-value project 2> /dev/null)" \
          --member serviceAccount:"$GKE_SERVICE_ACCOUNT_EMAIL" \
          --role roles/monitoring.viewer
          
      gcloud projects add-iam-policy-binding \
          "$(gcloud config get-value project 2> /dev/null)" \
          --member serviceAccount:"$GKE_SERVICE_ACCOUNT_EMAIL" \
          --role roles/monitoring.metricWriter
          
      gcloud projects add-iam-policy-binding \
          "$(gcloud config get-value project 2> /dev/null)" \
          --member serviceAccount:"$GKE_SERVICE_ACCOUNT_EMAIL" \
          --role roles/logging.logWriter

# GKE

from step 4

> create a GKE cluster

* 4.1, below is a cluster with node pool, and assign one one to each zone.

      export GKE_CLUSTER_NAME=istio-migration
      
      gcloud container clusters create "$GKE_CLUSTER_NAME" \
          --addons=HorizontalPodAutoscaling,HttpLoadBalancing \
          --enable-autoupgrade \
          --enable-network-policy \
          --enable-ip-alias \
          --machine-type=n1-standard-4 \
          --metadata disable-legacy-endpoints=true \
          --node-locations us-east1-b,us-east1-c,us-east1-d \
          --no-enable-legacy-authorization \
          --no-enable-basic-auth \
          --no-issue-client-certificate \
          --num-nodes=1 \
          --region us-east1 \
          --service-account="$GKE_SERVICE_ACCOUNT_EMAIL"

          [output]
          
          NAME             LOCATION  MASTER_VERSION  MASTER_IP      MACHINE_TYPE   NODE_VERSION  NUM_NODES  STATUS
          
      istio-migration  us-east1  1.11.7-gke.4    35.196.136.88  n1-standard-8  1.11.7-gke.4  3          RUNNING

# Istio Service

* 4.2, install Istion in GKE cluster using Helm CLI tool.

       export ISTIO_NAMESPACE=istio-system
       // using Namespace name for cluster, to init env variable
       
       kubectl apply -f "$ISTIO_PATH"/install/kubernetes/namespace.yaml
       // create Istion Namespace
       
       kubectl apply -f "$ISTIO_PATH"/install/kubernetes/helm/helm-service-account.yaml
       // this is a side service in Hell, called Tiller.
       
       "$HELM_PATH"/helm init --service-account tiller
       // install tiller in cluster.
       
       "$HELM_PATH"/helm install "$ISTIO_PATH"/install/kubernetes/helm/istio-init --name istio-init --namespace "$ISTIO_NAMESPACE"
       //Install the istio-init chart to bootstrap all Istio's custom resource definitions.
       
       [output]
       
            NAME:   istio-init
            LAST DEPLOYED: Wed Nov 18 13:15:12 2020
            
            NAMESPACE: istio-system
            STATUS: DEPLOYED
            
            RESOURCES:
            
            ==> v1/ClusterRole
            NAME                     AGE
            istio-init-istio-system  1s
            
            ==> v1/ClusterRoleBinding
            NAME                                        AGE
            istio-init-admin-role-binding-istio-system  1s
            
            ==> v1/ConfigMap
            NAME          DATA  AGE
            istio-crd-10  1     1s
            istio-crd-11  1     1s
            
            ==> v1/Job
            NAME               COMPLETIONS  DURATION  AGE
            istio-init-crd-10  0/1          1s        1s
            istio-init-crd-11  0/1          1s        1s
            
            ==> v1/Pod(related)
            NAME                     READY  STATUS             RESTARTS  AGE
            istio-init-crd-10-2s28z  0/1    ContainerCreating  0         1s
            istio-init-crd-11-28n9r  0/1    ContainerCreating  0         1s
            
            ==> v1/ServiceAccount
            NAME                        SECRETS  AGE
            istio-init-service-account  1        1s


