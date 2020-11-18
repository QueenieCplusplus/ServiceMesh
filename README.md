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

# WAF

from step 2 

> setup fire wall rule for both of the legacy and destination evn.

* 2.1, create a fire wall rule, and its options called desciption and action.

        gcloud compute firewall-rules create bookinfo \
              --description="Bookinfo App rules" \
              --action=ALLOW \
                --rules=tcp:9080,tcp:9081,tcp:9082,tcp:9083,tcp:9084 \
    --target-tags=bookinfo-legacy-vm
