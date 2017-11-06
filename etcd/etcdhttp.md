
#### Package = etcdhttp, Filename = client.go

An HTTP Server runs on the same machine as the EtcdServer and sits there listening for requests.  The main function of the etcdhttp package is to define the handlers,
and then the actual HTTP servers get fired
off inside the package **etcdmain** in the file *etcd.go*.

As you will see below
there are three HTTP handlers that take care of {keys, members, peers}

The **keysHandler** ServeHTTP method takes the request and calls
the method **Do** on the EtcdServer.  The Do method is one of the core
methods inside the EtcdServer interface defined in the package etcdserver,
filename server.go

```
func (h *keysHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  .
  .
  .
  resp, err := h.server.Do(ctx, rr)
```

This is the ONLY handler that calls the Do method.

Prior to this a new ClientHandler gets instantiated.

```
// NewClientHandler generates a muxed http.Handler
// with the given parameters to serve etcd client requests.
func NewClientHandler(server *etcdserver.EtcdServer) http.Handler {
```

There are 3 ServeHTTP methods inside the package **etcdhttp**

* keysHandler
* membersHandler
* peerMembersHandler

#### client.go

```
func (h *keysHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
func (h *membersHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
```

#### peer.go

```
func (h *peerMembersHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
```
