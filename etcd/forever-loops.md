
There are
two very important loops that run forever and get spawned off by
[goroutines]
(https://gobyexample.com/goroutines) in server.go

One of the **forever** loops run the Etcdserver and the other
loop runs Raft.

```
// This runs Etcdserver
go s.run()

// This runs Raft
go s.r.run()
```

Interestingly enough, the second routine gets spawned off by the first routine,
and the first routine gets run at the end of the start() routine which
fires up the Etcdserver.

For more details on the startup sequence go here.
