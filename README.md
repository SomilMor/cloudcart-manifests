\# CloudCart GitOps Manifests



This repository contains the Kubernetes deployment configuration for the \*\*CloudCart\*\* project.



It acts as the \*\*GitOps source of truth\*\* for the CloudCart workloads running on Amazon EKS.



The application source code and Jenkins CI pipeline are maintained separately in the `cloudcart-apps` repository.



\---



\## GitOps Workflow



\*\*cloudcart-apps → Jenkins → Docker Hub → cloudcart-manifests → Argo CD → Amazon EKS\*\*



The deployment process works as follows:



1\. Application code is pushed to the `cloudcart-apps` repository.

2\. Jenkins builds the four microservice images using Kaniko.

3\. Images are pushed to Docker Hub using the Jenkins build number as the image tag.

4\. Trivy scans the container images for HIGH and CRITICAL vulnerabilities.

5\. Jenkins updates the image tags in this repository.

6\. Jenkins commits and pushes the updated Kubernetes manifests.

7\. Argo CD detects the Git change.

8\. Argo CD automatically synchronizes the desired state with Amazon EKS.



\---



\## Repository Structure



```text

cloudcart-manifests/

|

|-- application.yaml

|

`-- k8s/

&#x20;   |

&#x20;   |-- ingress/

&#x20;   |   `-- ingress.yaml

&#x20;   |

&#x20;   |-- user/

&#x20;   |   |-- deployment.yaml

&#x20;   |   `-- service.yaml

&#x20;   |

&#x20;   |-- product/

&#x20;   |   |-- deployment.yaml

&#x20;   |   `-- service.yaml

&#x20;   |

&#x20;   |-- order/

&#x20;   |   |-- deployment.yaml

&#x20;   |   `-- service.yaml

&#x20;   |

&#x20;   `-- payment/

&#x20;       |-- deployment.yaml

&#x20;       `-- service.yaml

```



\---



\## Argo CD



`application.yaml` defines the CloudCart Argo CD application.



Argo CD monitors the `main` branch and recursively loads Kubernetes resources from:



```text

k8s/

```



Automated synchronization is enabled with:



```yaml

automated:

&#x20; prune: true

&#x20; selfHeal: true

```



This allows Argo CD to automatically deploy Git changes and correct configuration drift between Git and the Kubernetes cluster.



\---



\## Kubernetes Workloads



The repository manages four CloudCart microservices:



| Service | Kubernetes Service |

| --- | --- |

| User Service | ClusterIP |

| Product Service | ClusterIP |

| Order Service | ClusterIP |

| Payment Service | ClusterIP |



All application containers listen on port `3000`, while Kubernetes Services expose them internally on port `80`.



\---



\## Ingress Routing



External traffic enters the cluster through the NGINX Ingress Controller.



| Path | Service |

| --- | --- |

| `/users` | user-service |

| `/products` | product-service |

| `/orders` | order-service |

| `/payments` | payment-service |



The traffic flow is:



\*\*Client → AWS Load Balancer → NGINX Ingress → Kubernetes Service → Application Pod\*\*



\---



\## Automated Image Updates



Jenkins automatically updates the image field in each deployment after a successful build.



Example:



```yaml

image: somil7/user-service:39

```



The Jenkins build number provides a unique container version for each pipeline execution.



Jenkins then commits the new image tags to this repository, allowing Argo CD to perform the deployment.



\---



\## Related Repository



The application source code, Dockerfiles, Jenkins pipeline, observability configuration, architecture documentation, and project screenshots are available in the \*\*cloudcart-apps\*\* repository.



\---



\## Author



\*\*Somil Mor\*\*



DevOps and Cloud Enthusiast

