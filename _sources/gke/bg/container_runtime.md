# Container Runtime

**Read first**
- [The differences between Docker, containerd, CRI-O and runc](https://www.tutorialworks.com/difference-docker-containerd-runc-crio-oci/)
- [Kubernetes Containerd Integration Goes GA](https://kubernetes.io/blog/2018/05/24/kubernetes-containerd-integration-goes-ga/)

In summary, the container runtime used by nodes deployed in Google Kubernetes Engine (GKE) had moved from docker-based to containerd since Jun 2021 (`v1.19.12-gke.2100`). As a result, investigation methods (e.g. live response commands, mount container procedures) used for docker-based kubernetes clusters are no longer applicable for containerd-based clusters, and new / different methods have to be performed to obtain a similar outcome.

## Identification of Container Runtime
````{div} full-width
```shell
# with kubectl
kubectl get nodes -o wide
:' sample output
# For Docker runtime
NAME         STATUS   VERSION             OS-IMAGE                             CONTAINER-RUNTIME
gke-node-1   Ready    v1.16.15-gke.6000   Container-Optimized OS from Google   docker://19.3.1
gke-node-2   Ready    v1.16.15-gke.6000   Container-Optimized OS from Google   docker://19.3.1
gke-node-3   Ready    v1.16.15-gke.6000   Container-Optimized OS from Google   docker://19.3.1
# For containerd runtime
NAME         STATUS   VERSION           OS-IMAGE                             CONTAINER-RUNTIME
gke-node-1   Ready    v1.19.6-gke.600   Container-Optimized OS from Google   containerd://1.4.1
gke-node-2   Ready    v1.19.6-gke.600   Container-Optimized OS from Google   containerd://1.4.1
gke-node-3   Ready    v1.19.6-gke.600   Container-Optimized OS from Google   containerd://1.4.1
'

# with gcp cli
gcloud container node-pools list --cluster <cluster-name> --format="table(name,version,config.imageType)"
:' sample output
NAME          NODE_VERSION     IMAGE_TYPE
default-pool  1.19.6-gke.600   UBUNTU_CONTAINERD
default-pool  1.17.17-gke.9100 COS
'
```
````

## GKE Defaults
````{div} full-width
As of **30 Aug 2021 0250hrs (UTC)**
|Channel	|Version	|Default OS / Container Runtime	|Date Added	|
|---	|---	|---	|---	|
|Regular (default)	|1.20.8-gke.2100 (default)	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.20.9-gke.700	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|Rapid	|1.20.8-gke.2100 (default)	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.20.9-gke.2100	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.21.3-gke.901	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.21.3-gke.1100	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.21.3-gke.2000	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|Stable	|1.19.12-gke.2100 (default)	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.18.20-gke.901	|Container-Optimized OS with Docker (cos)	|	|
|	|1.18.20-gke.900	|Container-Optimized OS with Docker (cos)	|	|
|Static	|1.20.8-gke.2100 (default)	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.17.17-gke.9100	|Container-Optimized OS with Docker (cos)	|	|
|	|1.18.20-gke.900	|Container-Optimized OS with Docker (cos)	|	|
|	|1.18.20-gke.901	|Container-Optimized OS with Docker (cos)	|	|
|	|1.18.20-gke.2300	|Container-Optimized OS with Docker (cos)	|	|
|	|1.18.20-gke.3000	|Container-Optimized OS with Docker (cos)	|	|
|	|1.18.20-gke.3300	|Container-Optimized OS with Docker (cos)	|	|
|	|1.18.20-gke.4100	|Container-Optimized OS with Docker (cos)	|17 Jun 2021	|
|	|1.19.12-gke.2100	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.19.13-gke.700	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.19.13-gke.1200	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.19.13-gke.1900	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.20.8-gke.900	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.20.9-gke.700	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.20.9-gke.1100	|Container-Optimized OS with Containerd (cos_containerd)	|	|
|	|1.20.9-gke.2100	|Container-Optimized OS with Containerd (cos_containerd)	|	|
````