# https://hub.docker.com/r/itsthenetwork/nfs-server-alpine/
# docker build --rm -t nfs-server-alpine .
FROM alpine:3.4
MAINTAINER flytreeleft <flytreeleft@126.com>

RUN apk add -U -v nfs-utils bash iproute2 && \
    rm -rf /var/cache/apk/* /tmp/* && \
    rm -f /sbin/halt /sbin/poweroff /sbin/reboot && \
    mkdir -p /var/lib/nfs/rpc_pipefs && \
    mkdir -p /var/lib/nfs/v4recovery && \
    echo "rpc_pipefs    /var/lib/nfs/rpc_pipefs rpc_pipefs      defaults        0       0" >> /etc/fstab && \
    echo "nfsd  /proc/fs/nfsd   nfsd    defaults        0       0" >> /etc/fstab

# Build confd from source
## https://github.com/kelseyhightower/confd/blob/master/docs/installation.md#building-from-source-for-alpine-linux
#COPY bin/confd /usr/bin/confd
#COPY confd/confd.toml /etc/confd/confd.toml
#COPY confd/toml/* /etc/confd/conf.d/
#COPY confd/tmpl/* /etc/confd/templates/

COPY nfsd.sh /usr/bin/nfsd.sh
COPY bashrc /root/.bashrc

#RUN chmod +x /usr/bin/confd
RUN chmod +x /usr/bin/nfsd.sh

ENTRYPOINT ["/usr/bin/nfsd.sh"]
