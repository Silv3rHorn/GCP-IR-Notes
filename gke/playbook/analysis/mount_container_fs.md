# Mount Container Filesystem

* For the differences between `docker` and `containerd` container runtimes, and how to identify which is being utilised, pls refer to [Container Runtime](../../bg/container_runtime.md)

```{admonition} Pre-Requisites
* A forensic disk had been created from a snapshot of the Kubernetes node hosting the container
* Forensic disk had been attached to the analysis VM
* Mount directory had been created to mount the forensic disk filesystem - `/mnt/evidence/disk`
* Mount directory had been created to mount the container filesystem - `/mnt/evidence/container`
* Analyst has SSH access to the analysis VM
```

## docker container runtime

Uses [Google's docker-explorer](https://github.com/google/docker-explorer) and jq, which are recommended to be installed in [VM Instances for IR](../../../admin/vm_instances_for_ir.md)

```shell
# ssh into analysis vm
gcloud compute ssh <analysis_vm> --zone <zone>

# mount the disk of the compromised node
lsblk
sudo mount -o ro,noload /dev/<partition_id> /mnt/evidence/disk

# identify the compromised container
sudo de.py -r /mnt/evidence/disk/var/lib/docker list all_containers | jq '.[].image_name'
# obtain the container id
sudo de.py -r /mnt/evidence/disk/var/lib/docker list all_containers | jq '.[] | select(.image_name | contains("<keywords>"))'

# mount the container
sudo de.py -r /mnt/evidence/disk/var/lib/docker mount <container_id> /mnt/evidence/container
cd /mnt/evidence/container
```
* * *

## containerd

### Container Explorer

Uses Google's [container-explorer](https://github.com/google/container-explorer), recommended to be installed in [VM Instances for IR](../../../admin/vm_instances_for_ir.md)

```shell
# ssh into analysis vm
gcloud compute ssh <analysis_vm> --zone <zone>

# mount the disk of the compromised node
lsblk
sudo mount -o ro,noload /dev/<partition_id> /mnt/evidence/disk

# identify the compromised container
sudo container-explorer -i /mnt/evidence/disk \
  --support-container-data /usr/bin/supportcontainer.yaml \
  list containers

# mount the desired container
sudo container-explorer -i /mnt/evidence/disk \
  --support-container-data /usr/bin/supportcontainer.yaml -n <namespace> \
  mount <container_id> \
  /mnt/evidence/container
  
# mount all containers (except support containers)
# mounting all containers will create sub-directories using container ID as directory name
sudo container-explorer -i /mnt/evidence/disk \
  --support-container-data /usr/bin/supportcontainer.yaml \
  mount-all /mnt/evidence/container

# unmount all under /mnt/evidence/container
sudo umount $(grep /mnt/evidence/container /proc/mounts | cut -f2 -d" " | sort -r)
sudo rm -rf /mnt/evidence/container/*
```

Support Containers according to container-explorer documentation are:
* When a GKE cluster is created, several containers are created to support the Kubernetes
* These clusters are used to support Kubernetes only and may not be interesting for the investigation.
* Kubernetes support containers are hidden by default when the global flag `--support-container-data=supportcontainer.yaml` is used
    * `supportcontainer.yaml` contains the commonly known hostname, image, and labels used to identify the support containers
* When `--support-container-data` is used, the `list` and `mount-all` commands automatically ignore the known support containers where applicable
* You can use `--show-support-containers` and `--mount-support-containers` to display and mount the support containers.

### Legacy

**At the Kubernetes node:**
````{div} full-width
```shell
# identify the compromised container
crictl ps --all

# get the container id

# get the mount command of the container
mount | grep -i <container id>
:' sample output (formatted for clarity)
overlay on \
  /run/containerd/io.containerd.runtime.v2.task/k8s.io/2e634bd0729e2880fedd159da4f8988bb69e840a1bcae7a84ccfe1bd85432430/rootfs \
  type overlay (rw,relatime,lowerdir= \
  /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/154/fs: \
  /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/153/fs: \
  /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/152/fs: \
  /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/151/fs: \
  /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/150/fs: \
  /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/149/fs, \
  upperdir= \ 
  /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/155/fs, \ 
  workdir= \ 
  /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/155/work)
'
```
````

* From the sample output, we can see that
    * `overlay` mount type is used
    * `lowerdir` consists of the following snapshot layers
        * `149` - `154`
    * `upperdir` consists of the following snapshot layers
        * `155`
    * `workdir` consists of the following snapshot layers
        * `155`

**At the analysis VM:**
````{div} full-width
```shell
# ssh into analysis vm
gcloud compute ssh <analysis_vm> --zone <zone>

cd ~

# mount the disk of the compromised node
lsblk
sudo mount -o ro,noload /dev/<partition_id> /mnt/evidence/disk

# mount the container filesystem (command is based on the sample output above)
sudo mount -t overlay overlay -o ro, 
lowerdir= \ 
/mnt/evidence/disk/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/155/fs: \ 
/mnt/evidence/disk/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/154/fs: \
/mnt/evidence/disk/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/153/fs: \
/mnt/evidence/disk/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/152/fs: \
/mnt/evidence/disk/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/151/fs: \
/mnt/evidence/disk/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/150/fs: \
/mnt/evidence/disk/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/149/fs \
/mnt/evidence/container
```
````

* From the mount command, we can see that
    * `upperdir` and `workdir` parameters are not used (since it is mounted as read-only)
    * `lowerdir` references the exact same snapshot layers from the sample output of our `mount` command on the **Kubernetes Node**, except that is pre-pended with the original upper snapshot layer `155`
    *  mount destination is `/mnt/evidence/container`

