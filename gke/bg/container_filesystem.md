# Container Filesystem

- 2 types of filesystem are used in the container world
    * Overlay filesystem
        * e.g. `AUFS` and [Overlayfs](#overlayfs)
        * Multiple directories with file diffs for each layer in an image
        * Usually work on common filesystem types such as EXT4 and XFS
    - Snapshotting filesystem
        * e.g. [Snapshotters](#snapshotters), `devicemapper`, `brtfs` and `ZFS`
        * Handle file diffs at the block level
        * Only run on volumes formatted for them
- Different filesystems had been observed to be used by each [container runtime](./container_runtime.md) in GKE
    - docker-based runtime - [Overlayfs](#overlayfs)
    - containerd runtime - [Snapshotters](#snapshotters)

## Identification of Container Filesystems

```shell
# determine storage driver (for docker runtime only)
docker info | grep -i storage
:' sample output 
Storage Driver: overlay2
'

# determine filesystem (for docker runtime only)
docker info | grep -i filesystem
:' sample output
Backing Filesystem: extfs
'

# determine snapshotter type (for containerd runtime only)
crictl info | grep-i snapshotter
:' sample output
"snapshotter": "overlayfs",
'
```

## Overlayfs

Refer to Docker's documentation [here](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)

## Snapshotters

```{note}
Following is a summary translation of [this mandarin chinese blog](https://blog.frognew.com/2021/06/relearning-container-09.html)
```

* Following lines are seen in containerd’s configuration file
```shell
root = /var/lib/containerd
state = "/run/containerd"
```
* `root` - Used to store persistent data, such as 
    * content
    * snapshot
    * metadata
    * runtime
```shell
tree /var/lib/containerd/ -L 2
/var/lib/containerd/
├── io.containerd.content.v1.content
│   ├── blobs
│   │   └── sha256
│   └── ingest
├── io.containerd.metadata.v1.bolt
│   └── meta.db
├── io.containerd.runtime.v1.linux
├── io.containerd.runtime.v2.task
├── io.containerd.snapshotter.v1.btrfs
├── io.containerd.snapshotter.v1.native
│   └── snapshots
├── io.containerd.snapshotter.v1.overlayfs
│   ├── metadata.db
│   └── snapshots
│       ├── 1
│       ├── 2
│       ├── 3
│       ├── 4
│       ├── 5
│       └── 6
└── tmpmounts
```

* Each of the sub-directories corresponds to the plugins indicated in `ctr plugin ls`
    * Essentially, these sub-directories are used by containerd plugins to store data
    * Each plugin has its own directory
    * containerd’s storage capabilities are realised through plugins (e.g. snapshotter)
    * Pulled images are stored in `io.containerd.content.v1.content/blobs/sha256`
        * Each sub-directory corresponds to either
            * an index file - view with `cat`
            * a manifest file - view with `cat`
            * a config file
            * a layer file - decompress with `tar`
    * containerd decompresses layer file contents to `io.containerd.snapshotter.v1.overlayfs/snapshots` directory
        * Each container image can contain multiple layers
        * Each decompressed layer corresponds to a sub-directory in `snapshots`
    * Main purpose of `snapshotter` plugin is to mount each layer to prepare `rootfs`
        * Corresponds to `graphdriver` in Docker
        * Default snapshotter is [Overlayfs](#overlayfs)
        * snapshotter turns read-only image layers to `lower` directory
        * snapshotter turns read-write image layers to `upper` directory
        * End result is `merged` directory located in `/run/containerd/io.containerd.runtime.v2.task/k8s.io/<container_id_long>/rootfs`
        * Use `mount | grep /var/lib/containerd` to determine which snapshot directories are mounted as `lower` and `upper`