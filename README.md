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

