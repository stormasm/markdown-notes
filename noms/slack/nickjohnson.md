
### 2017

nickjohnson [5:39 AM]
joined #general

nickjohnson [5:40 AM]
Hello Noms folks. I'm from the go-ethereum team, and we're looking at alternate datastores, both for geth itself and for applications built on top of the blockchain.

[5:40]
A versioned datastore like noms is a good choice, because reorgs in a blockchain can cause you to need to change to different versions of history at any time.

[5:41]
A couple of questions, though:
- Do you have any benchmark data? How much overhead does noms have over leveldb in typical use?
- How far away is garbage collection, reasonably speaking?

rafael [1:46 PM]
hi, @nickjohnson, welcome. exciting to see you here!

[1:47]
benchmarks: there are some encoding benchmarks here: https://github.com/attic-labs/noms/tree/master/go/perf/codec-perf-rig
GitHub
attic-labs/noms
noms - The versioned, forkable, syncable database


[1:47]
but that doesn't say much about IO (benchmarks read/write to in-memory store).

[1:48]
for the next stable version of noms (master is currently unstable), LDB will be deprecated and replaced with a block storage layer which is similar in some ways to LDB but better suited to noms usage (edited)

[1:49]
NBS: https://github.com/attic-labs/noms/tree/master/go/nbs
GitHub
attic-labs/noms
noms - The versioned, forkable, syncable database


[1:50]
i can imagine that many of the motivations for noms to prefer nbs would also apply to you.

[1:52]
GC I'd characterize as being medium-term. It's work that hasn't started yet, but much of the groundwork is in place and we'll need compaction (which includes GC) for a number of reasons.

[1:52]
It's hard to say anything concrete about timing, but I'd be somewhat surprised if there wasn't at least rudiments of this in place by EOY

rafael [2:00 PM]
As to the question about overhead, it's kind of hard to answer in the abstract. Maybe you can say something about a characteristic usage you'd be considering.


----- March 20th -----
besquared [6:36 PM]
joined #general. Also, @oter joined.


----- March 24th -----
joeblew99 [3:33 AM]
the GraphQL API looks really nice. I was wondering if i can use it in an embedded scenario.  By “embedded" i mean having golang GUI client call direct into graphql, which then calls into the NOMS db. Nothing over the wire. I would be using NOMS connection over the file system (since its all embedded.

[3:34]
It would mean that i could also use the same code by call over the wire to the server too. So te code is kind of network topology agnostic.

[3:35]
sorry about askng this, but i have looked at the example and code, and could not really see a definitive example or deduct it from the code. So i figured that its worth asking the team direct.

rafael [4:12 PM]
@joeblew99: if you asking if you can make an in-process call directly to GraphQL/noms, the answer is yes. You could basically call: https://github.com/attic-labs/noms/blob/master/go/ngql/query.go#L70
GitHub
attic-labs/noms
noms - The versioned, forkable, syncable database


[4:12]
It'll write the JSON response to a writer that you hand in.

new messages

----- March 27th -----
smithb [11:10 AM]
joined #general


----- March 28th -----
aa [10:16 AM]
@nickjohnson seconded that it's nice to see you here. awhile back i read about the "patricia trees" used in eth, and they are similar in motivation to noms' prolly trees.

[10:16]
https://github.com/attic-labs/noms/blob/master/doc/intro.md#prolly-trees-probabilistic-b-trees
GitHub
attic-labs/noms
noms - The versioned, forkable, syncable database


[10:21]
the main difference iirc is that prolly trees give b-tree like perf over any key, regardless of its distribution. which we needed in order to build database-style indexes.

[10:23]
anyway, i'd enjoy hearing more about the problems you're trying to solve. is there any background you can point me to? alternately, consider starting an issue in noms github to discuss.

nickjohnson [10:47 AM]
Cool - can prolly trees be cryptographically verified?

[10:47]
So, I've got two usecases in mind.

[10:49]
First is applications that build on top of the blockchain. Presently, most just watch some block several blocks back from the head, on the theory that reorgs that far back are unlikely, and treat it like a standard data feed. But big reorgs _can_ happen, and this also leads to latency. A versioned database could do away with this: you just insert data as you get it, versioned by the head block, and any time there's a reorg you just play the new data into a new version from the fork point; easy, and no more worrying about reorgs.

[10:50]
The second is actually storing go-ethereum's data. Presently we store the merkle-patricia trie directly in leveldb, and look up elements by looking up stuff in the tree - but this requires a leveldb lookup for each node, so we're effectively building indexes inside indexes. We could write trie nodes directly to disk, but their branching factor is so low, we'd end up doing a lot of small disk lookups, which is likewise inefficient (though at least a factor-of-logn improvement over presently)

[10:51]
But a third option is that we could materialise the values at the leaves and store those (in addition to the nodes, since we need them for some network operations) in a versioned database, which would allow us to look up data with a single db lookup instead of many.

[10:51]
So those are the things that bring me to considering noms - and separately, stratified b-trees.

aa [11:17 AM]
@nickjohnson prolly trees can be verified in the sense that a given logical value, no matter how large, always has a specific hash. it doesn't matter the sequence of operations that led to the value. the hash for a given value is deterministic.

nickjohnson [11:20 AM]
And by 'logical value' you mean an entire tree/index?

aa [11:20 AM]
yes

nickjohnson [11:20 AM]
And I assume this property applies recursively (I can't think of any way it wouldn't)?

aa [11:20 AM]
really the entire database.

[11:20]
yep.

nickjohnson [11:21 AM]
Interesting. That would have made a nice match for Ethereum. I think it's unlikely we'll ever get a chance to change now, though.

[11:23]
Also, stupid question - can noms be run entirely in-process, and backed by a local filesystem?

aa [11:23 AM]
yes, that is sort of the way it works on one level.

[11:23]
it's an embedded go database.

[11:23]
if you check out noms head right now and follow the samples, it will direct you to type file paths on the cli

nickjohnson [11:23 AM]
I thought as much

[11:23]
okay

aa [11:24 AM]
those file paths will end up creating databases on disk

[11:24]
those databases our our "noms block store" format

[11:24]
which replaced our usage of ldb

nickjohnson [11:24 AM]
For geth itself, it looks like nbs might be a better match than noms itself

[11:24]
While noms would be an appropriate backend for ethereum-based applications

aa [11:25 AM]
can i ask some questions about your comments above

nickjohnson [11:25 AM]
Absolutely

aa [11:25 AM]
i'm sorry, i know how ethereum works at an intellectual level, but i don't really have it loaded into ram if you know what i mean

nickjohnson [11:25 AM]
Yup

aa [11:26 AM]
"A versioned database could do away with this: you just insert data as you get it, versioned by the head block, and any time there's a reorg you just play the new data into a new version from the fork point; easy, and no more worrying about reorgs."

[11:26]
is the problem that the app has its own app-specific data

[11:27]
which sort of goes with or is built on specific blocks? (edited)

nickjohnson [11:27 AM]
Pretty much. Let me give you a concrete example.

[11:28]
We have a contract that implements a naming system, which is tree structured. Onchain, the actual tree relationship isn't explicitly stored, but whenever a new tree node is created, that fact (the parent and child node IDs) is logged as an event on chain

[11:28]
By playing back all those node creation events, you could reconstruct a view of the entire tree - and that's something an application would reasonably want to do.

[11:28]
But reorgs make that more complicated, since you could play back some events that then later cease to exist due to a reorg.

[11:29]
(Events are accessible to listeners, but not to code running 'onchain', and aren't guaranteed to be stored indefinitely by normal nodes)

[11:30]
The same goes for any app that wants to respond to events that happen onchain, or to display to the user a representation of things that have happened on chain and are still relevant.

[11:30]
But in general the pattern of a sort of streaming-mapreduce based on chain events is quite common - couchdb meets blockchain, of a kind.

aa [11:31 AM]
ok

[11:31]
but you generally need to unpack it into some more useful structure

[11:31]
you -> the app

nickjohnson [11:31 AM]
Yup, exactly

aa [11:32 AM]
does the naming app have any state that isn't reproducible from the data in the blockchain?

[11:33]
(is off chain data important for me to understand right now?)

nickjohnson [11:33 AM]
In the case of the naming app, no. Other apps might, but it seems likely those wouldn't need to be versioned along with the blockchain.

aa [11:33 AM]
ok

[11:34]
i see. yeah. cool problem.

[11:34]
so i will try to repeat the problem back:

[11:35]
problem: some contracts implement complicated datastructures that can't be represented onchain

[11:35]
instead they sort of record mutation events on the chain

[11:35]
given a head block they need to be able to reconstruct their index or whatever

nickjohnson [11:36 AM]
Yup. Though the "they" that need to be able to reconstruct the index is external users, not code on the chain.

aa [11:37 AM]
does "code on the chain" translate to "contract" ?

[11:37]
:slightly_smiling_face:

nickjohnson [11:37 AM]
Yup

aa [11:37 AM]
yeah so i mean you could have a noms database that maps blocks to whatever your datastructure is.

nickjohnson [11:38 AM]
It's a generally encouraged pattern - if a contract doesn't need to retrieve the data, but external users will, emit an event and forget about it instead of paying for (expensive) data storage. So in a token contract, for instance, it stores the current balance, but not a ledger of all past transactions.

[11:38]
Yup, that's what I'm thinking - just create a version for each block.

aa [11:38 AM]
then when you get a new head, you go find the last block you knew about

[11:38]
then rebuild from there

nickjohnson [11:38 AM]
*nods*

[11:38]
And that means that the code on top of it can be a simple map/reduce style indexer, with no need for handling reversions, 'unreduce' or any such complication.

aa [11:40 AM]
ok, other question

[11:40]
"The second is actually storing go-ethereum's data. Presently we store the merkle-patricia trie directly in leveldb, and look up elements by looking up stuff in the tree"

[11:41]
um, sorry for dumb question, but remind me again what the mp trie is storing

[11:41]
and what you're looking up in it?

nickjohnson [11:41 AM]
All the data in the Ethereum state is stored in a merkle patricia trie whose root is provided in each block on the chain

[11:42]
(Actually, that's the root of the 'accounts trie', which indexes accounts by their addresses, mapping to (balance, nonce, code, storage_trie_root). Each account can also have a 'storage trie' which works the same way but for contract storage)

[11:42]
So any time you want to look up an account balance, or a contract's data, you look it up in a trie

aa [11:43 AM]
gotcha

[11:43]
how do you map that to ldb?

nickjohnson [11:44 AM]
Presently we use LDB as a content-addressed store (at least, for trie nodes -we store a few other things there, mostly but not all content-addressed)

[11:44]
Each trie node contains the hashes of its child nodes, and so forth (edited)

aa [11:45 AM]
this gets out of my area of expertise, you should really talk to @rafael

[11:45]
he is the primary one behind nbs (edited)

nickjohnson [11:45 AM]
I suspect nbs to be a good match, though since it only supports a single root, I suspect we'd have to build our own index structure on it to store the multiple roots we need

aa [11:45 AM]
but the main thing wiht ldb is that you are paying for updatabilitity

[11:45]
there's a lot of complexity there

nickjohnson [11:45 AM]
Yup. And like you, we mostly only update the roots.

aa [11:46 AM]
noms supports multiple "datasets"

[11:46]
which are really the updatable thingies from user pov

[11:46]
but they are labels into a graph that has a single root

nickjohnson [11:46 AM]
So you must have some kind of copy-on-write btree for the dataset roots?

aa [11:46 AM]
all of noms is a gigantic copy on write btree

[11:47]
well prolly tree

[11:47]
the root of noms is a noms Map

nickjohnson [11:47 AM]
(Speaking of, have you read about stratified btrees?)

[11:47]
I've been pondering implementing them myself in Go, because they sound pretty damn cool.

aa [11:47 AM]
@rafael looked at them way back at the beginning of noms

[11:48]
i can't remember anything more than that :slightly_smiling_face:

nickjohnson [11:48 AM]
righto

[11:48]
The authors' main criticism of COW B-Trees is that they require a lot of node writes per storage update, and that reads are random rather than sequential, so don't benefit from locality (though that latter is less important on an SSD)

aa [11:49 AM]
yeah nbs doesn't yet, but plans to implement a strategy for re-localziing

nickjohnson [11:49 AM]
But I'm not so sure what you're doing is that far from it; from a quick skim of the code it looks like you're already writing and merging tables in a similar fashion to leveldb

aa [11:49 AM]
this is the "compaction" that raf referred to yesterday

nickjohnson [11:49 AM]
ah

[11:49]
And does NBS implement garbage collection yet? Or is that also compaction? :slightly_smiling_face:

aa [11:50 AM]
gc will basically be a side effect of compaction

nickjohnson [11:50 AM]
makes sense (edited)

rafael [11:50 AM]
catches up

aa [11:51 AM]
i asked raf to poke his head in. i think he is more the one to talk about the second use case. but in general we've been pretty happy with the wins that are available from throwing updates overboard.

nickjohnson [11:51 AM]
Good to know, thanks.

aa [11:51 AM]
i would love to see an ethereum app try to use noms for the first use case.

[11:51]
it seems really easy to try out.

[11:51]
there are no external deps, it's just straight go.

nickjohnson [11:52 AM]
The name service thing I mentioned is a real use-case that I'm working on. I'm also tempted to build a couchdb-style layer on top of noms so external users can define indexes based on events from the chain.

[11:52]
Since that's a really common way to set up events and contracts.

aa [11:53 AM]
we are building a thing that is similar to that.

[11:53]
it's a natural way to want to work on these systems.

[11:53]
it's not oss yet tho

nickjohnson [11:53 AM]
Yeah, and it's nice to solve the issues couchdb had with deleting items by using proper versioning (at least in our use-case)

[11:53]
okay

aa [11:54 AM]
i'm really excited to see this happen. lmk what i can do to help.

nickjohnson [11:54 AM]
Will do!

rafael [12:03 PM]
ok. caught up.

[12:04]
yeah, i did look at and consider stratified B-Trees when we started.

[12:04]
I think looking back on it now, NBS is a better fit for our use case for a few reasons:

[12:05]
1) stratified B-trees are trying to strike a balance between data duplication and read amplification which is specifically bounded

[12:05]
IOW, it's trying to offer guarentees about performance *across* versions.

[12:06]
there's a kind of no-free-lunch issue here. you can spend a bunch of storage and offer lower read amplication for all versions, or you can disadvantage some versions and save space (and complexity)

[12:06]
for noms, we generally feel like typically you want the *head* to be fast and only occasionally care about old revisions.

nickjohnson [12:07 PM]
Hm. To me, though, one of the big advantages of stratified btrees was the reduction in write amplification - for a regular COW btree you need to write O(log n) nodes for every write.

[12:07]
Or does NBS implement a leveldb-like structure, where each level is its own btree?

rafael [12:08 PM]
well, there's a related issue here:

[12:08]
sorry, back up. (edited)

[12:09]
in noms, you will need to eventually write O(log n) new nodes because it's a COW tree.

[12:10]
however, NBS takes inspiration from LDB in the sense that writes are *deferred*

[12:11]
meaning that you do a bunch of writing, which is buffered in memory and then periodically is compacted to disk.

nickjohnson [12:11 PM]
Gotcha. But you're still updating a single btree, rather than writing a new one per level?

rafael [12:11 PM]
notably, because NBS is CA storage, not k/v storage, it can defer has-checking of already contained blocks to the point when it compacts to disk.

[12:11]
which is not possible with LDB

[12:12]
sorry. don't understand the question.

nickjohnson [12:13 PM]
I'm just comparing with the approach stratified btrees take, which is to build a separate btree in each table they write out to disk

[12:14]
The idea being that then you reduce the number of writes required (at the cost of merging n parallel btrees for an n level database when doing queries)

[12:14]
Which is similar to leveldb, though I don't recall if it builds btrees in its levels or just orders them

rafael [12:16 PM]
i see. yes, noms is just updating a single p(b)tree

[12:16]
my memory of stratified b-trees is a bit fuzzy, but i know that ldb only stores runs of k/v pairs

nickjohnson [12:17 PM]
*nods*

[12:17]
Regarding using noms for geth - I assume you have some data structure you use to store multiple mutable roots in it? Is it anything we could reuse?

[12:17]
Generally we need to be able to look up content by its hash, and also have a limited number of roots we need to look up by other keys

[12:18]
(Actually, they're not all mutable, but there's, eg, a list of block numbers we need to map to block hashes)

rafael [12:19 PM]
right now NBS stores CA byte sequences and exactly ONE variable which *should be* hash of the root block.

[12:19]
the root hash is stored in the manifest as opposed to a table.

[12:20]
this is important so that the semantics of table storage (where CA blocks are stored) are consistent

[12:20]
(only operations are insert, g/c)

nickjohnson [12:20 PM]
Yup, this much I'd gathered

[12:20]
But I assume you implement a COW-tree of some kind at the root in order to have more than one 'mutable' reference?

rafael [12:21 PM]
yes, that happens at the noms level.

[12:21]
as @aa mentioned, the data model is the following:

[12:21]
the root hash is a `Map<String, Ref<Commit>>`

[12:22]
commits retained references to history.

nickjohnson [12:22 PM]
Makes sense. Are these data structures accessible to applications outside noms, so we don't have to write our own?

rafael [12:24 PM]
Noms type system is pretty simple. All apps interact with these types: `Map, Set, List, Blob` (these types are automatically chunked into p-trees), `Number, String, Bool` (Primitives), `Union, Ref`

[12:25]
`Ref` represents a typed reference to a value which is stored in a distinct (addressable) block

aa [12:25 PM]
@nickjohnson if you embed noms into a go app, these "types" are the apis you will talk to noms in terms of

[12:25]
(drive-by clarification)

nickjohnson [12:25 PM]
Right; but I'm talking here about using NBS directly, rather than noms itself.

rafael [12:26 PM]
Are you asking if you can use noms to create p-trees which you store yourself in NBS?

[12:27]
as opposed to letting noms manage the storage of blocks?

nickjohnson [12:28 PM]
Pretty much

[12:28]
I want to know if I can use part of noms as a library for our own purposes with NBS.

rafael [12:28 PM]
i see. yeah, theoretically.

[12:28]
that usage hadn't occurred to me.

[12:29]
there might be some rough edges, but i don't think there's anything preventing it.

nickjohnson [12:29 PM]
Since our merkle-patricia trie nodes already implement a versioned trie, there doesn't seem a need to store it in noms - but a fast content-addressed store is perfect.

[12:29]
But we still need more than one root in order to be able to look blocks up by their numbers, etc.

rafael [12:30 PM]
i see.

[12:30]
i don't see any reason why not.

nickjohnson [12:30 PM]
Okay, now the big pain-in-the-ass question: Is there any way to swap out the hash function used by NBS?

rafael [12:31 PM]
curious: can you saw more about this use?

[12:31]
i.e. the block lookup by number?

nickjohnson [12:32 PM]
Oh - it's used in a number of places in the Ethereum interface, including in user-facing APIs

[12:32]
Most people just want block 123456 from the 'main chain', whatever that happens to be

[12:32]
So we maintain an index mapping from block number to hash, and update that when there's a reorg

rafael [12:34 PM]
i see. and how would you expect to represent that within another root of nbs?

nickjohnson [12:36 PM]
I'm imagining a situation where the root of NBS is the root of a tree of 'tables'/'datasets'. Some of those are just root hashes (eg, the current most recent block on the main chain); one would be 'numbers', say, which is itself the root of an index tree mapping block numbers to block hashes.

rafael [12:37 PM]
i see. and this number increases monotonically as the main chain extends?

nickjohnson [12:37 PM]
Yup

[12:38]
Also, we occasionally need to rewrite references to recent block numbers if there's a reorg

rafael [12:38 PM]
right. makes sense.

[12:38]
so, yeah. about the hash function.

[12:38]
at one point it was entirely provided by the embedder

[12:39]
how many bytes is your hash?

[12:39]
that'd be the biggest problem. i think the code assumes 20-byte hashes (we use the first 20 bytes of sha256)

[12:40]
looking at the code now

[12:42]
right. yeah, the code right now is indirectly dependent explicitly on our hash, but it not a very tight coupling.

nickjohnson [12:42 PM]
It's a 20 byte hash, fortunately

rafael [12:43 PM]
heh. nice.

[12:43]
in that case, it's probably not too big a deal to abstract it.

nickjohnson [12:44 PM]
Okay, that's promising.

rafael [12:44 PM]
about the root tree of `tables/datasets`, i think there a number of options.

[12:46]
there's no requirement that a dataset retain history.

[12:46]
you're free to essentially replace the single commit record at each logical commit

[12:47]
i.e. if you wanted to use a ptree (noms `Map`) to maintain efficient mapping of numbers to block address

nickjohnson [12:47 PM]
*nods*

[12:48]
For the root, our list of datasets is small enough we could probably just store a single mapping in a blob at the root

[12:48]
We'd need a tree of some sort for the block number mapping, since that's about 3 million entries and growing.

rafael [12:49 PM]
the way prolly trees work, there isn't much benefit to doing that.

[12:49]
meaning if it's a small set, then it almost always be encoded in a single block (chunk)

nickjohnson [12:50 PM]
Makes sense

rafael [12:50 PM]
also, blobs are chunked with the same frequency, so you'd have similar chunking behavior but lack semantic support for the map

[12:51]
fwiw, our experience is the following:

[12:52]
we've hit a bunch of cases where we got cute because we thought we were worried about chunking doing the wrong thing and we unwound almost all of those and have arrived at generally just letting chunking do its thing because it works really well.

nickjohnson [12:52 PM]
Okay. If those are primitives we can use ourselves on top of NBS, that sounds perfect.

rafael [12:53 PM]
sweet.

nickjohnson [12:55 PM]
Incidentally, have you done any performance profiling on NBS as the number of keys stored grows in size?

[12:55]
It'd be interesting to see inserts/second plotted against database size for it and leveldb

rafael [12:55 PM]
well, it's not really a fair fight right now

[12:55]
ldb does alot of compaction under heavy write load

[12:56]
that dominates long-term throughput.

[12:56]
nbs doesn't currently do any.

nickjohnson [12:56 PM]
good point

aa [12:57 PM]
Meaning we crush then

[12:57]
Them*

rafael [12:57 PM]
really the thing that caused us to start thinking about nbs was because inserting O(10 gig) of structured data became unesuably slow, largely because of compaction) (edited)

aa [12:57 PM]
:-)

rafael [12:59 PM]
but to answer your question, we've profiled nbs perf inserting data in the 10s of gig range

[1:00]
the only thing right now that grows with size of data stored is the time it takes to "dedup" pre-existing chunks as memtables are compacted to disk

[1:01]
however, since memtable compaction is async and bulk has checks only very rarely go to disk, the effect isn't especially meaningful

[1:02]
fwiw, if you plan to use the noms data structures and store data in nbs, managing your own "core" DAG offers obvious flexibility, but you (might) loose a number of potentially useful things: (edited)

[1:04]
- syncing (just like git -- but faster, and about to get even more fast =-) (edited)

[1:04]
- usage of a supported CLI (e.g. diff, log, etc...)

[1:05]
- consistency & validity checking

nickjohnson [2:19 PM]
When you say 10s of gigs - is that one 10G object, or 10 million 1MB objects?

[2:19]
We're not worried about syncing, though the other two might be useful. (edited)

[2:19]
Any idea how much overhead using all of noms adds on top of using just the datastore?

rafael [2:29 PM]
10 gig of data that is chunked into blocks which are on average 4K. (edited)

nickjohnson [2:32 PM]
Ah, righto

rafael [2:38 PM]
Again, overhead is hard to characterize broadly. I'd say that io overhead is extremely minimal.

[2:39]
Relative to what I assume your path of using noms data structures and nbs, I can't think of significant CPU overhead.


----- March 29th -----
nickjohnson [12:53 AM]
I think the main overhead for using noms for geth would be that it would be building an index on top of NBS for our mostly-already-content-addressed storage (all our blocks)

[12:53]
Whereas if we use NBS directly we can store that data, which makes up the bulk of our storage, directly in NBS

rafael [10:55 AM]
I don't have a clear picture in my head of what you are imagining, but we're happy to do what we can to help.
