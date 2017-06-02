Source repository of [itsthenetwork/nfs-server-alpine](https://hub.docker.com/r/itsthenetwork/nfs-server-alpine/),
but [confd](https://github.com/kelseyhightower/confd) was removed to keep the mini size of image.

## Overview

A handy NFS Server image comprising of;

- [Alpine Linux](http://www.alpinelinux.org/) v3.4.0. Alpine Linux is a security-oriented,
  lightweight Linux distribution based on [musl libc](https://www.musl-libc.org/) (v1.1.15)
  and [BusyBox](https://www.busybox.net/).
- NFS v4 only, over TCP on port 2049.

Before running, we should build the image first:

```bash
sudo bash ./build.sh
```
or

```bash
sudo docker build --rm -t itsthenetwork/nfs-server-alpine .
```

When run, this container will make whatever directory is specified by the environment
variable `SHARED_DIRECTORY` available to NFS v4 clients.

```bash
docker run -d --name nfs --privileged \
           -v /some/where/fileshare:/nfsshare \
           -e SHARED_DIRECTORY=/nfsshare \
       itsthenetwork/nfs-server-alpine:latest
```

Add `--net=host` or `-p 2049:2049` to make the shares externally accessible via the host networking stack.
This isn't necessary if using [Rancher](http://rancher.com/) or linking containers in some other way.

Due to the `fsid=0` parameter set in the `/etc/exports` file,
there's no need to specify the folder name when mounting from a client.
For example, this works fine even though the folder being mounted and shared is `/nfsshare`:

```bash
sudo mount -v 10.11.12.101:/ /some/where/here
```

To be a little more explicit:

```bash
sudo mount -v -o vers=4,loud 10.11.12.101:/ /some/where/here
```

To unmount:

```bash
sudo umount /some/where/here
```

The exports file contains these parameters:

```
*(rw,fsid=0,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)
```

Note that the `showmount` command won't work against the server as rpcbind isn't running.

## RancherOS

You may need to do this to get things working;
```bash
sudo ros service enable kernel-headers
sudo ros service up kernel-headers
```

RancherOS also used overlayfs for Docker so please read the next section.

## OverlayFS

OverlayFS does not support NFS export so please volume mount into your NFS container
from an alternative (hopefully one is available).

On RancherOS the `/home`, `/media` and `/mnt` file systems are good choices as these are ext4.

## Other OS's

You may need to ensure the `nfs` and `nfsd` kernel modules are loaded by running `modprobe nfs nfsd`.

## Mounting Within a Container

The container requires the `SYS_ADMIN` capability, or, less securely, to be run in `privileged` mode.

```bash
docker run -d --name nfs --cap-add=SYS_ADMIN \
           -v /some/where/fileshare:/nfsshare \
           -e SHARED_DIRECTORY=/nfsshare \
       itsthenetwork/nfs-server-alpine:latest
```

**NOTE**: In Ubuntu, the extra option `--security-opt apparmor:unconfined` is needed.

## What Good Looks Like

A successful server start should produce log output like this:

```
Starting Confd population of files...
Displaying /etc/exports contents...
/nfsshare *(rw,fsid=0,async,no_subtree_check,no_auth_nlm,insecure,no_root_squash)

Starting NFS in the background...
rpc.nfsd: knfsd is currently down
rpc.nfsd: Writing version string to kernel: -2 -3 +4
rpc.nfsd: Created AF_INET TCP socket.
rpc.nfsd: Created AF_INET6 TCP socket.
Exporting File System...
exporting *:/nfsshare
Starting Mountd in the background...
```
