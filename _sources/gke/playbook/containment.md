# Containment

## Isolate Compromised Workloads

* Move uncompromised workloads and future deployment of workloads to other nodes
* Objective: Reduce an adversary’s ability to impact other workloads on the same node

```shell
# cordon node with compromised workload to ensure no other pods are scheduled on it
kubectl cordon <node_name>

# label the pod that was compromised
kubectl label pods <pod_name> quarantine=true

# drain the node of pods that are not labeled with quarantine
kubectl drain <node_name> --pod-selector='!quarantine'
```
* * *

## Restrict Network Traffic

* To prevent lateral movement or communication with adversary-controlled infrastructure

### Network Policy Level

```{note}
This can only be used if Network Policy is enabled for the cluster (disabled by default in GKE). To determine if it had been manually enabled, run the command `gcloud container clusters describe <cluster_name> —zone <zone> —project <project> | grep -i "networkpolicy:" -a3`.

**DO NOT** enable Network Policy for the cluster after the incident had occurred. This would drain, delete, and recreate all nodes in all node pools of the cluster. As a result, all temporary storage volumes (including evidence) will be deleted
```

* A deny all traffic rule may help stop an attack that is already underway by severing all connections to the pod
* The following Network Policy, when implemented, will apply to all pods with the label `app=nginx`
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: default-deny
spec:
    podSelector:
        matchLabels: 
            app: nginx
    policyTypes:
    - Ingress
    - Egress 
```
```shell
# save the above policy to a yaml file

# apply network policy
kubectl apply -f <yaml_file>

# verify network policy had been applied
kubectl get networkpolicies
```

```{warning}
It takes ~5mins for the applied Network Policy to take effect.
```

### Service Level

* Delete / Modify associated services
    * `LoadBalancer` service exposes the service externally using GCP network load balancer
    * `ClusterIP` service exposes the service on a cluster-internal IP
        * Not effective as the default `kubernetes ClusterIP` service will be automatically created
    * `NodeIP` service exposes each node in the cluster using a specific port
    * `ExternalName` service maps the service to the contents of the `externalName` field (e.g. `foo.bar.example.com`), by returning a `CNAME` record with its value
```shell
# list all services
kubectl get services
        
# either delete service OR
kubectl delete service <service_name>

# modify selector of associated service
kubectl patch service <service_name> -p '{"spec":{"selector":{"<key>": "<value>"}}}'
```

### Network Isolate the Nodes Hosting the Compromised Pods

* Prevents new outbound connections to other nodes in the cluster using an egress rule
* Prevents inbound connections to the compromised node using an ingress rule
* Allows SSH inbound connections to the compromised node from defined IP only

```shell
# tag the compromised node to which the firewall rule would be applied
gcloud compute instances add-tags <node_name> --zone <zone_name> --tags quarantine

# create fireawall rule to deny all egress TCP traffic from instances with the quarantine tag
gcloud compute firewall-rules create quarantine-egress-deny \
    --network <network_name> \
    --action deny \
    --direction egress \
    --rules tcp \
    --destination-ranges 0.0.0.0/0 \
    --priority 0 \
    --target-tags quarantine
    
# create a firewall rule to deny all ingress TCP traffic to instances with the quarantine tag
# use priority of 1, which allows overriding with another rule that allows SSH from a specified IP
gcloud compute firewall-rules create quarantine-ingress-deny \
    --network <network_name> \
    --action deny \
    --direction ingress \
    --rules tcp \
    --source-ranges 0.0.0.0/0 \
    --priority 1 \
    --target-tags quarantine
    
# allow SSH from specific IP address where investigation would be conducted from
# use priority of 0, which would override the previous rule to deny all ingress TCP traffic
gcloud compute firewall-rules create quarantine-ingress-allow \
    --network <network_name> \
    --action allow \
    --direction ingress \
    --rules tcp:22 \
    --source-ranges <ip_address> \
    --priority 0 \
    --target-tags quarantine
```

### Remove External IP Addresses of Nodes Hosting the Compromised Pods

* Breaks any existing network connections outside the VPC
* **NOT RECOMMENDED** as may self-heal and re-configure another external IP address (most likely a different one)

```shell
# find the access config that associates the external IP with the node
gcloud compute instances describe <node_name> \
    --zone <zone_name> --format="flattened([networkInterfaces])"

# look for the lines that contain name and natIP. They look like the following:
networkInterfaces[0].accessConfigs[0].name:   ACCESS_CONFIG_NAME
networkInterfaces[0].accessConfigs[0].natIP:  EXTERNAL_IP_ADDRESS

# find the value of natIP that matches the external IP you want to remove.
# note the name of the access config

# remove the external IP
gcloud compute instances delete-access-config <node_name> \
    --zone <zone_name> --access-config-name <access_config_name>
```
* * *

## Pause/Delete/Restart Pods

* It is **NOT POSSIBLE** to **pause** containers (or pods) in Kubernetes, like how it is possible with `docker pause` command
* **NOT RECOMMENDED** as terminating pods would destroy pod storage to varying levels, which would make analysis impossible
    * docker runtime (Graph-Driver; overlay2) - All pod storage deleted
    * containerd runtime (Snapshotters; overlayfs) - Read-Only snapshot layers (already available from the original container image) are preserved, but Read-Write snapshot layers are deleted
* Pods can be terminated after Live Response had been performed and all necessary evidence had been collected
* If the to-be-deleted pod is managed by a higher-level Kubernetes construct (e.g. `Deployment`, `DaemonSet`, `ReplicaSet`), deleting the pod **schedules a new pod** (which run new containers)
    * Requires deletion of the high-level construct to delete the pod and prevent re-scheduling

```
# delete pods
kubectl delete pod <pod_name>
kubectl delete pod -l <selector>

# delete high-level construct
kubectl delete <high_level_construct> <name>
# alternatively, if part of deployment, scale replicas down to 0
kubectl scale --replicas=0 deployment.apps/<deployment_name>

# restart a specific container within the pod
kubectl exec -it <pod_name> -c <container_name> -- /bin/sh -c "kill 1"

# stop container after SSH into node
# docker container runtime
docker stop <container_id>  # send SIGTERM signal and wait 10 seconds for process to exit, if not exited send SIGKILL signal
docker kill <container_id>  # send SIGKILL signal immediately
docker rm -f <container_id>  # stop and remove container
# containerd
crictl stop <container_id>
crictl rm -f <container_id>
```
