
Etcd is a model Golang system for many reasons.

All Golang systems should be as easy to build, test and deploy as Etcd.

To build, test and deploy Etcd one simply executes the following 3 commands.

```
./build
./test
./bin/etcd
```

Etcd is based on Raft, a consensus algorithm which allows state machines to
remain consistent over long periods of time.  Typically computer systems
crash and need to be rebooted, raft enables a smooth transition as things
happen in the cloud.

A major motivation for studying and learning Etcd is that its a great
introduction to Golang; especially for beginners to the language who want
to understand a real world system.  It has all of the interesting components
that are important for a developer to understand prior to launching full
force into the language.  If you are new to Golang, Etcd is a great place
to get your feet wet.

The goal of this book is to first give you a high level view of Etcd, and
then for those more curious to dive in deeper. This is a work in progress
just like Etcd.  As Etcd evolves and matures this book will stabilize over
time.  For now, Etcd is moving quickly and so will this book.
