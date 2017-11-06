We will begin with a high level view of the startup phase of Etcd.  The
details will emerge further down on the page.

Two major things happen in the startup phase of Etcd.

* Raft is setup to accept requests from its peers
* The http server and client infrastructure need to be initialized

There are 2 major lines of communication happening at the same time.

* The clients, you for example, send in requests to Etcd.
* The followers need to be able to communicate with the leader.

Both of these transports are based on http.  But the neat thing about
the design of Etcd is in the future they don't have to be.

#### Startup Sequence

* main.go

```
func main() {
	etcdmain.Main()
}
```

* package etcdmain, file etcd.go

```
func Main() {
  .
  .
  .
  startEtcd(cfg)
```

startEtcd launches the etcd server and HTTP handlers for client/server communication.

```
// Start prepares and starts server in a new goroutine. It is no longer safe to
// modify a server's fields after it has been sent to Start.
// It also starts a goroutine to publish its server information.
func (s *EtcdServer) Start() {
	s.start()
	go s.publish(defaultPublishRetryInterval)
	go s.purgeFile()
}
```

This is where the first forever loop gets spawned.

```
func (s *EtcdServer) start() {
	if s.snapCount == 0 {
		log.Printf("etcdserver: set snapshot count to default %d", DefaultSnapCount)
		s.snapCount = DefaultSnapCount
	}
	s.w = wait.New()
	s.done = make(chan struct{})
	s.stop = make(chan struct{})
	s.stats.Initialize()
	// TODO: if this is an empty log, writes all peer infos
	// into the first entry
	go s.run()
}

```

The second forever loop which starts up Raft on each individual machine gets
launched in the first forever loop see below.


```
func (s *EtcdServer) run() {
	snap, err := s.r.raftStorage.Snapshot()
	if err != nil {
		log.Panicf("etcdserver: get snapshot from raft storage error: %v", err)
	}
	confState := snap.Metadata.ConfState
	snapi := snap.Metadata.Index
	appliedi := snapi
	// TODO: get rid of the raft initialization in etcd server
	s.r.s = s
	s.r.applyc = make(chan apply)
	go s.r.run()
	defer close(s.done)
```

The documentation for the second forever loop in Raft is documented
[here in the Raft package]
(http://godoc.org/github.com/coreos/etcd/raft).
