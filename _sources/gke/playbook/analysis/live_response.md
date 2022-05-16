# Live Response

```{note}
Commands listed are mainly for triaging. Commands to perform more in-depth investigations are excluded.
```

* Container OS is typically minimal and many native binaries are not available
* Some kubernetes management CLI (`kubectl`) commands are required to fill this gap

## kubectl

```{note}
**Append `kubectl` commands below with `--namespace <namespace>` to filter resource by namespace or `--all-namespaces` to include all namespaces**
````

```shell
# list all pods
kubectl get pods --all-namespaces

# get details of pod
kubectl describe pods <pod-name>

# get resource usage of all pods
kubectl top pod

# get resource usage of all nodes
kubectl top node

# get date of container
kubectl exec <pod-name> [-c <container-name>] -- date

# get bash history
kubectl exec <pod-name> [-c <container-name>] -- find / -iname '.bash_history'
kubectl exec <pod-name> [-c <container-name>] -- cat <bash history location>

# get contents of /etc/passwd
kubectl exec <pod-name> [-c <container-name>] -- cat /etc/passwd

# get contents of /etc/hosts
kubectl exec <pod-name> [-c <container-name>] -- cat /etc/hosts

# get network connections of container
# warning: installing netstat might destroy evidence
kubectl exec <pod-name> [-c <container-name>] -- apt update
kubectl exec <pod-name> [-c <container-name>] -- apt install net-tools
kubectl exec <pod-name> [-c <container-name>] -- netstat -pano

# get routing table
kubectl exec <pod-name> [-c <container-name>] -- netstat -nr

# get network interfaces
kubectl exec <pod-name> [-c <container-name>] -- ifconfig -a

# get container logs
kubectl logs <pod-name> -c <container-name>

# copy files from container to host
kubectl cp <namespace>/<pod-name>:<src-file> <dst-file> -c <container-name>

# copy files from host to pod
kubectl cp <src-file-path> <namespace>/<pod-name>:<dst-file-path>

# get a shell
kubectl exec <pod-name> [-c <container-name>] -- /bin/sh
```
* * *

## docker cri

```shell
# list all containers
docker ps -a

# get details of container (cmd, image, volumes, mounts)
docker inspect <container-id>

# get date of container
docker exec -it <container-id> date

# get storage location of container
docker inspect <container-id> |  grep -i GraphDriver -A8

# get running processes of container
docker top <container-id> -eo user,pid,ppid,stime,command

# get changes to files or directories on a container's filesystem
docker diff <container-id>

# get bash history
docker exec -it <container-id> find / -iname '.bash_history'
docker exec -it <container-id> cat <bash history location>

# get contents of /etc/passwd
docker exec -it <container-id> cat /etc/passwd

# get contents of /etc/hosts
docker exec -it <container-id> cat /etc/hosts

# get network connections of container
# warning: installing netstat might destroy evidence
docker exec -it <container-id> /bin/sh
apt update
apt install net-tools
netstat -pano

# get routing table
docker exec -it <container-id> netstat -nr

# get network interfaces
docker exec -it <container-id> ifconfig -a

# get container logs
docker logs <container-id>

# copy files from container to host
docker cp <container-id>:<src-file> <dst-file>
```

### Using docker-forensics

```shell
git clone https://github.com/kk0m4k/docker-forensics.git
cd docker-forensics
mv config.json.example config.json
sudo python df.py -i <container-id>
```
* * *

## containerd

* `crictl` commands are more limited than `docker` commands
* `kubectl` commands can be used to fill the gap

```shell
# list all containers
crictl ps -a

# get details of container (image, volumes, mounts)
crictl inspect <container-id>

# get date of container
crictl exec -it <container-id> date

# get stats of container
crictl stats <container-id>

# get bash history
crictl exec -it <container-id> find / -iname '.bash_history'
crictl exec -it <container-id> cat <bash history location>

# get contents of /etc/passwd
crictl exec -it <container-id> cat /etc/passwd

# get contents of /etc/hosts
crictl exec -it <container-id> cat /etc/hosts

# get network connections of container
# warning: installing netstat might destroy evidence
crictl exec -it <container-id> /bin/sh
apt update
apt install net-tools
netstat -pano

# get routing table
crictl exec -it <container-id> netstat -nr

# get network interfaces
crictl exec -it <container-id> ifconfig -a

# get container logs
sudo crictl logs <container-id>

# copy files from container to host
crictl cp <container-id>:<src-file> <dst-file>
```
