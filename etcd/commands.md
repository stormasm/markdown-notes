
[How to get all of the keys in etcd](https://github.com/etcd-io/etcd/issues/5323)

ETCDCTL_API=3 etcdctl get "" --prefix=true
ETCDCTL_API=3 etcdctl get "" --from-key
