
#### Testing

In the process of figuring out the details of how Etcd works one is faced
with a lot of things happening inside Etcd in a very short period of time.
To better understand how Etcd works I like putting *fmt.Println* at different
spots in the code to better understand the flow and timing of when things
happen.

One of the things that will get in your way of understanding the flow is
that a SYNC happens on a regular basis.

```
package = etcdserver, filename = server.go

// sync proposes a SYNC request and is non-blocking.
// This makes no guarantee that the request will be proposed or performed.
// The request will be cancelled after the given timeout.

replace this:

func (s *EtcdServer) sync(timeout time.Duration) {
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	req := pb.Request{
		Method: "SYNC",
		ID:     s.reqIDGen.Next(),
		Time:   time.Now().UnixNano(),
	}
	data := pbutil.MustMarshal(&req)
	// There is no promise that node has leader when do SYNC request,
	// so it uses goroutine to propose.
	go func() {
		s.r.Propose(ctx, data)
		cancel()
	}()
}

with this:

func (s *EtcdServer) sync(timeout time.Duration) {}

```

To turn this off temporarily simply comment out completely all of the code
except for the method signature allowing the sync to continue to be called
with no effect.  After your debugging and understanding is completed then
simply turn it back on.
