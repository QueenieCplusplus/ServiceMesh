# ServiceMesh
Network Architecture of Migrating Microservices of App from On-premise to Cloud

using Istio Mesh to connect and monitor those microsevices in a secure way.

![](https://cloud.google.com/solutions/images/supporting-your-migration-with-istio-mesh-expansion-service-mesh.svg)

# Core Steps:

(1) access the legacy (on-premise) enviroment.

(2) provision the target enviroment.

(3) config a service mesh.

(4) add service in (1) into (3).

(5) deploy services in (3).

(6) split traffic by setting up Routing Rule.

(7) route traffice to (3).

(8) retire (1).

