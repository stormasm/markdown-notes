
Data comes into Raft from multiple sources.

There are internal messages that get generated to keep raft in **SYNC**,
and there are external messages that come in via client requests.

In either case, the package raft, **Node** interface has a method called Propose
*which proposes that data be appended to the log.*

This method is called in two places in etcdserver:server.go

* In the sync method
* In the Do method

The Do method gets called by the client any time it wants to add a new
key to Etcd.  The sync method gets called internally by Etcd.
