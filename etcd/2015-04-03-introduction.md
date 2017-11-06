---
layout: post
title:  "Introduction"
date:   2015-04-03
categories: etcd raft
---

We are excited to be writing about Etcd from a programmer's point of view.  In
this blog we will outline all of the different facets of Etcd as the program
evolves over time.  Meaning, as Etcd advances and makes progress in its natural
evolution as a project, we will address and comment on the changes that are taking
place as they happen.  So stay tuned :)

At the heart of Etcd is Raft.  Talking about the different components that
talk to Raft will be one of the themes of this blog.

If you are developing a new Raft system its possible that you may want to use
many of the components that are already built into Etcd besides just the Raft
package.

So lets take a look at the packages and see what we can leave out of the pure
Raft implementation.

So our assumption is the following for this introduction:

*We are going to treat the raft package as a black box, meaning we don't
need to know anything about raft at the moment, except some basic concepts.*

These basic concepts are:

* we need a way to save state to disk
* we need a way to talk to the system
* we need some type of a controller

My argument here is that there is a one to one correspondence between
these three concepts and the underlying packages needed to fulfill our
desire to accomplish these tasks.

#### Wal

Wal stands for **write ahead log**.  It is the Etcd package that is used to save the state
to disk.  Prior to the state being saved to disk, it is stored temporarily in a log
file and then once the coast is clear, that data gets written to disk.  The coast being
clear is a reference to knowing that all of the nodes have to have the data prior to
persisting the state machine.

Etcd chose to write its own persistent store rather than using an off the shelf system
like
[redis]
(http://redis.io/)
or
[bolt]
(https://github.com/boltdb/bolt)

Both of these systems are probably fast enough to work just fine.  Bolt could
be used without even firing up another process, as its an in memory database
similar to
[Sqlite]
(https://www.sqlite.org/)

But the **wal** system is simple and fast enough to work just fine too, and
so if you are developing a Raft system, wal would be a fine option.  Plus,
all of the code to integrate into Raft has already been written and the concepts
and code base is being tested by everyone who uses Etcd.  So in my mind, it
makes sense to use wal.
