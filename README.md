# notes-on-gundb #

a (growing) collection of notes on GunDB

![Network of Pearls](img/network-of-pearls.png)
<br><span style="font-size:10px">(image made with [DiffusionBee](https://github.com/divamgupta/diffusionbee-stable-diffusion-ui) and [Realistic Vision v2.0](https://civitai.com/models/4201/realistic-vision-v20))</span>

[GunDB](https://github.com/amark/gun) is a decentralized (peer-to-peer) distributed [graph database](https://en.wikipedia.org/wiki/Graph_database) with support for multiple users, access control and persistence. It runs in the browser or on [Node.js](https://nodejs.org/en), works fine when offline and synchronizes with other peers using a "last-write-wins" strategy when online.

While GunDB itself may be brilliant in many aspects, the docs are a complete nightmare (hard to read, chaotic, incomplete, wrong). This repository therefore contains a collection of the author's notes on GunDB written while he tries to learn and use it.

In the end, these notes will hopefully be less chaotic than the official docs or the GunDB wiki.

> feel free to [open an issue](https://github.com/rozek/notes-on-gundb/issues) if you find errors in my descriptions

## Table of Contents ##

Just as a hint: you can see an outline of these notes and directly navigate to a specific section by clicking on the "list" icon left of the "README.md" title above the "notes-on-gundb" header - or at the top of this window in case that you already had to scroll down!

## GunDB Documentation and Chat ##

**Do not use `https://gun.eco/docs` - use the [GunDB wiki](https://github.com/amark/gun/wiki) instead, it's much better**. The descriptions in this collection will therefore refer to the wiki rather than to the "official" docs.

If you have questions and don't find any answers in the docs, the wiki or these notes, you may try joining the [chat on GunDB](https://app.gitter.im/#/room/#amark_gun:gitter.im). 

## Basic Concepts and Terms ##

GunDB is a decentralized (peer-to-peer) distributed graph database.

### Database Structure ###

The entries in this database are called **nodes**. Every node has a globally unique **id** (called its **soul**) and may contain an arbitrary number of **properties**. Properties may contain **values** (either `null` or a boolean, number or string) or **links**. Links are pointers to nodes (identified by their soul) and may either point to _other_ nodes or (directly or indirectly) to themselves: <u>circular refe ja rences are deliberately permitted!</u>

At the beginning, the only known node is the **root node**. By (recursively) following the links in this node (or directly navigating to a given soul) other nodes can be visited and their properties loaded. The set of all nodes is sometimes also called the **universe** of a graph database.

Nodes directly "above" the root node play a special role, within these notes they are therefore called **trunk nodes** and all remaining nodes **branch nodes**.

### Users ###

In GunDB, **users** are represented by _cryptographic key pairs_. The <u>public</u> key of such a pair is used to identify a user within the GunDB universe and to provide a trunc node for her/him. The <u>private</u> key of that pair is required in order to get the permission to write into the trunc node and above.

As indicated by their name, _private_ keys should be kept safe and not shared: when lost, the related user space can no longer be written to - when leaked, everybody with that key may modify the information in the related user space. On the other side, _public_ keys may deliberately be shared - they allow others to navigate to the related user space, verify a user's signatures or encrypt messages for that user.

Since cryptographic keys look like long random numbers, users may also be referenced by a (human readable) **alias**. When created, the trunc node for such an alias contain the public key for that user and can be used to navigate to the actual user space.

It should be noted that _everybody may define new users_ for GunDB just by creating a cryptographic key pair (and the corresponding trunc node) or by creating a new alias. Furthermore, "users" do not necessarily have to represent human beings - they could also represent user _groups_, data spaces for applications, chat rooms and much more. 

### Topology ###

The participants of a graph database are called **peers**. At the moment, GunDB provides JavaScript _clients_ for such peers, these may be used by (human-controlled) applications, relays or other automated systems.

Data transfer takes place between these peers and does not require any central servers for that purpose (GunDB is a **peer-to-peer** database). However, in order to establish such a data transfer, the other peers have to be found first - that's what **relay peers** are good for: **business peers** contact one or multiple relay peers in order to be informed about the location of any required nodes and then communicate with those locations directly. This takes load off individual relays and also reduces the probability of database failure by simply moving to another relay, should one of them shut down.

Similarly, peers that have expressed interest in certain nodes obviously (temporarily or permanently) hold a copy of these nodes. This knowledge may be exploited by other peers by requesting such nodes from any of those peers that are known to hold copies - further reducing both the load on individual peers and the probability of an overall failure.

### Synchronization ###

While connected, any changes made by any peer are immediately reported to any other peer which needs this data.

Nevertheless, a network outage may always disconnect some peers from the rest of a GunDB database - this is called **network partitioning**. While GunDB in itself remains usable, remote data may not be requested, remote changes not observed and local changes not propagated to other peers - the so called **split brain problem**. However, as soon as the network becomes operational again, all changes may be delivered and the nodes held by individual peers **synchronized** with the rest of the database. As long as the delayed reported changes do not affect the same properties of the same nodes, synchronization remains simple - in case of a **conflict**, however, GunDB uses some **metadata** for the affected properties stored along in the same nodes to implement a **last-write-wins** strategy, where later changes overwrite former ones.

### Eventual Consistence ###

Principally, the clients of a graph database never know if the data they received so far is complete and up-to-date - there could always remain one last peer which stores the final bit of information to make the image complete, but which has unfortunately been disconnected from the rest of the network until now.

By design, graph databases can only guarantee that their contents become **eventually consistent**, i.e., that they synchronize as soon as disconnected peers get connected again. On the other side, however, this restriction allows any client to keep working with the database even in the case of a "failure" (e.g., a network partition) as there is no central server which could be "down".

As a consequence, the clients of a graph database should always listen to (and react on) incoming changes of the nodes they have requested before. Graph databases are **dynamic databases**.

### Data Removal ###

By design, nodes and their contents cannot easily be deleted - there could always be another node that did not receive the deletion report but still holds a copy of the outdated node and sends it back to the database again. Instead, old nodes and properties have to be **nulled out** in order to be marked as **garbage**. This operation creates a **tombstone** which is kept in the database and eventually overwrites the original data in every peer. If it becomes necessary to actually get rid of no longer wanted nodes, some kind of **distributed garbage collection** has to be implemented on top of the graph database - GunDB does not (yet) provide such a DGC.

### Scalability ###

By design, graph databases may grow as much as desired: an unlimited number of peers may work with an unlimited number of users and an unlimited number of nodes which itself may contain an unlimited number of properties (like the root node often does). Because the nesting of nodes is also unlimited, their souls could become longer and longer without end - although this situation should better be avoided as there is no optimization mechanism for this case (unnecessary nodes and properties may easily just be ignored by any peer, but a node's soul can not)

### Privacy ###

In an open peer-to-peer distributed system there is by design no "privacy" - at first. There are, however, mechanisms (based on cryptography) which can provide some protection: **digital signatures** prevent others from writing into a given user space (unless explicit **certificates** allow them to do so), **(symmetric) encryption** protects property values from being read by others, and **asymmetric encryption** provides for the exchange of information between explicitly given users.

Nevertheless, in order to allow for synchronization in a peer-to-peer network, the souls of nodes and the names of node properties always have to be kept readable for others - only property _values_ can be encrypted. While the encryption of links may be technically feasible, doing so would break navigation using the GunDB API: if you really have to encrypt your links, you will also have to implement your own function to follow them.

Thus, when modelling your database, always keep in mind that the contents of any node is principally visible for everybody. Even if you encrypt node property values, some metadata remains visible (namely the souls and property names of your nodes) - you will have to obfuscate them as well, if you want to make spying more difficult.

### Persistence ###

Nodes requested during runtime are persisted locally in every peer (browsers persist in localStorage or IndexedDB as far as their quota permits, Node.js clients persist on the file system) - but more reliable backups may also be made with the help of automated peers that try to fetch every node, e.g., in a given user space.

### Usage ###

GunDB uses **callbacks** not unlike **event handlers** to inform an application about incoming changes on nodes that are under observation.

It goes very well with **reactive programming** frameworks based on **observables** (sometimes called **signals**) which, when changed, automatically trigger updates of a user interface at the required places (or invoke any other dependent function)

### Data Mapping ###

Business data rarely fits directly to a GunDB node as the contents of such a node are limited to `null`, boolean, number or string values and links. GunDB neither supports arrays nor nested objects - even if these are used to hold **tuples** and **structures** whose elements are not meant to be changed individually but only as a whole. Furthermore, business objects are mostly implemented as **instances** of **classes** rather than **plain objects**. As a consequence, the use of GunDB normally requires the implementation of an **interface layer** which maps nodes to business data and vice-versa.

### Data Modelling ###

A graph database contains nodes and links. Each node is individually addressable and has to be fetched separately - this should be taken into account when designing the mapping between business data and GunDB nodes. Items of business objects which always belong to these objects should probably be better kept within "their" object (unless size constraints enforce a separate node). On the other hand, items used in multiple places of a business application can naturally be stored in an own node.

## Preparational Steps ##

GunDB can be used directly in the browser without having to "build" (or "pack") an application before (this case is sometimes called a **no-build environment**). As a consequence, it is sufficient to simply add the following elements to the `<head>` section of a web page:

```
<script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/sea.js"></script>
```

You may then use one of the global variables `Gun` or `GUN` directly. (For other variants - such as the usage as an ECMAScript module (ESM) or the usage within Node.js please consider the [Wiki](https://github.com/amark/gun/wiki/Getting-Started-(v0.3.x)))

Now, add another `<script>` element (either to the `<head>` as well or to the `<body>` section) with your own code:

```
<script>
  const Gun = new GUN()
  
  ;(async () => {
    ...
  })()
</script>
```

The (perhaps) strange looking `async` [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) is not strictly necessary, but it allows you to use the [`await` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) within your code in order to wait for completion of asynchronous GunDB methods.

## Contexts ##

GunDB's **fluent API** is based on the concept of contexts: many methods (such as `get`, `put`, `on` and others) return a **context** object which can then be used by the following method. Context objects are a bit like "handles" representing one (or multiple) database nodes - albeit with additional information that can be used to navigate back to the node that was addressed before.

It is important to understand that the node(s) represented by a given context do not have to exist in the database - if necessary, they will be created as soon as some data is written to them.

`Gun` itself represents the root node, `Gun.get(soul)` a node with the given soul, and `context.get(subpath)` a node with the given subpath relative to the given context (see section [Paths](https://github.com/rozek/notes-on-gundb#pathse) for an explanation how the actual node id is built).

Context objects have the following structure (only the most important properties are shown, consider the [Wiki](https://github.com/amark/gun/wiki/GUN%E2%80%99s-Data-Format-(JSON)) for more details):

```
{
  _ : {
    $: ...,      // points to the current context object
    root.$: ..., // if present, refers to the root context object
    back.$: ..., // if present, refers to the previous context object
    soul: ...,   // if present, contains the id of the represented node
    link: ...,   // if present, contains the id of the represented node
    get: ...,    // if present, contains the "key" of the represented node
    put: ...,    // if present, contains the known contents of the...
                 // ...represented node including any metadata
  }
}
```

When concatenating method calls in a **chain**, some methods depend on the context returned from the previous call, some don't - the [Wiki](https://github.com/amark/gun/wiki/Chaining-(v0.3.x)) contains a table which shows the individual dependencies.

## Paths ##

When chaining `get` calls, GunDB usually concatenates the individually given arguments with a slash (`/`) in between to build the id of the final node. The outcome looks not unlike the path names used in file systems and gives the illusion(!) of a "containment tree". However, it is important to understand that there is no such tree: every node is independent of any other - and even nodes with an id like `a/b/c` linking to other nodes with ids of the form `a/b/c/d` (or similar) do not _contain_ the nodes they link to!

```
  const Context_1 = Gun.get('a').get('b').get('c')
    Context_1.put({ message:'hi!' })  // node must exist for this study
    await waitFor(100)                // allow "put" to settle
  console.log(Context_1._.link || Context_1._.soul) // displays "a/b/c"

  const Context_2 = Gun.get('a/b').get('c')
  console.log(Context_2._.link || Context_2._.soul) // displays "a/b/c"

  const Context_3 = Gun.get('a/b/c')
  console.log(Context_3._.link || Context_3._.soul) // displays "a/b/c"

  const Context_4 = Gun.get('a').get('b/c')
  console.log(Context_4._.link || Context_4._.soul) // displays "undefined"
```

The first three variants of `get` finally address the same node (`waitFor` is explained [below](https://github.com/rozek/notes-on-gundb#waitfor))

> As a "rule of thumb" you could expect that the argument of the _first_ `get` method applied to the root node context (i.e., `Gun.get(...)`) may contain slashes and still works as expected - all other `get` calls should avoid arguments with slashes...(yes, the GunDB API is often quite unsystematic)

To drive the illusion further, operations writing nested objects to a given node automatically create new nodes for the nested objects with souls built from the concatenation of the original node's id and the name of the property with the nested object:

```
  const Context_1 = Gun.get('TestObject')
    Context_1.put({ nested: {message:'hi!'} })    // writes a nested object
  console.log(await PayloadOf(Context_1)) // displays an object with a link

  const Context_2 = Gun.get('TestObject/nested') // was built automatically
  console.log(await PayloadOf(Context_2))     // displays "{message:'hi!'}"
```

(`PayloadOf` is explained [below](https://github.com/rozek/notes-on-gundb#PayloadOf))

## Working with Nodes and Properties ##

### Addressing Nodes ###

As mentioned above, nodes may be addressed using the [get](https://github.com/amark/gun/wiki/API-(v0.3.x)#get) method:

```
  const Context_1 = Gun.get('a').get('b').get('c')
  const Context_2 = Gun.get('a/b/c')
```

> Nota bene: a node does not have to exist in order to be addressed

* an attempt to address a node with an empty id (`''`) will log(!) the error message "0 length key!" (rather than throwing an exception)
* node ids may contain control characters (even `\0`)
* node ids may contain any number of slashes
* node id `~` is permitted
* node id `~@` will log(!) an error _object_ containing the error message "Alias not same!" (rather than throwing an exception)
* node ids of the form `~@xxx` will also log(!) such an error _object_ unless the client has previously been authenticated using the alias following the `~@`
* node ids of the form `~xxx` work fine unless the character sequence following the `~` looks like a public key and the client has not been authenticated using a key pair containing that public key: in such a case an error _object_ with the error message "Unverified data." is logged(!) (rather than throwing an exception)
* there does not seem to be an explicit limit for node ids (I tried 10k long ids which worked)

> **Conclusion: for professonal application development it will be extremely important to provide a wrapper around the original API which throws exceptions instead of logging useless messages which do not even contain any tracebacks**

### Reading Nodes ###

`once` can be used to retrieve (a snapshot of) the contents of a given node - including its actual payload and some metadata.

```
  async function ContentsOf (Context) {
    return new Promise((resolve,reject) => {
      Context.once((Contents) => {
        resolve({...Contents})               // "Contents" must not be modified!
      })
    })
  }

  const Context_1 = Gun.get('TestObject')
    Context_1.put({ nested: {message:'hi!'} })
  console.log(await ContentsOf(Context_1))

  const Context_2 = Gun.get('an/unknown/node')
  console.log(await ContentsOf(Context_2))
```

If a node exists, the object returned from `once` will contain a property `_` (with the node's metadata) and all other properties the node holds at the moment (if any). If a node does not exist, the returned object will be empty.

> Nota bene: if preferred, you may also extend GunDB with a method which returns the actual payload of a node given by its context, as shown [below](https://github.com/rozek/notes-on-gundb#contextdata)

### Following Links ###

If a property of a given node looks like a link (i.e., it is an object with the single property `#` which contains a string), then the (always absolute) id of the link target may be extracted and used as an argument for `Gun.get()` in order to address the target node.

```
  let NodeData = await Gun.get('a/node').Data()
  let TargetNode = Gun.TargetOfLink(NodeData.link)
// assuming that 'a/node' has a property 'link' which is a node
```

> nota bene: these lines use the author's GunDB extensions [Data](https://github.com/rozek/notes-on-gundb#contextdata) and [TargetOfLink](https://github.com/rozek/notes-on-gundb#guntargetoflink)

### Working with Node Properties ###

The properties of a node given by its context can be written with `put`, providing a plain JavaScript object containing a key-value pair with "key" being the name of the property and "value" the value to be written:

```
  Gun.get('an/existing/node').put({ 'property-name':'value' })
```

Missing properties will be created, existing ones overwritten - regardless of their previous value. Other properties of the given node are left untouched. Allowed values are

* `null`,
* boolean, number or string primitives,
* nested objects,
* links or
* node contexts (see below)

By convention, writing `null` into a property marks it as "garbage" and creates a "tombstone" for it.

If a given node does not yet exist, it will be created on-the-fly:

```
// assume that 'a/missing/node' does not yet exist
  Gun.get('a/missing/node').put({ 'property-name':'value' })
// the node will exist now
```

An attempt to assign a (nested) object to a property will create an additional node and write a link to that node into the property. The new node will have an id which consists of the original node's "soul", a slash (`/`) and the name of the property receiving the link:

```
  Gun.get('an/outer-node').put({ 'inner-node':{ data:'will be created' } })
// will create 'an/outer-node/inner-node' with the sole property 'data'
```

> **Dangerous**: an attempt to write an array into a property will log(!) the error message "Invalid data: Array at ..." (instead of throwing an exception). Trying to read the contents of the addressed node may lock the application (the `once` callback will never be called)

An attempt to assign an object of the form `{ '#':'...' }` (i.e., a link) to a property will principally write the given link. However,

* an attempt to write an empty link `{ '#':'' }` will instead write a link to a node with an id consisting of the id of the original node, a slash `/` and the name of the property written to (this _could_ be an undocumented feature, though)

An attempt to write a GunDB context object into a node property will write a link to the addressed node instead:

```
  Gun.get('a/node').put({ link:Gun.get('other/node') })
// will write a link to 'other/node' into property 'link'
```

This behaviour is independent of whether the target node exists or not.

> **Conclusion: for professonal application development it will be extremely important to harden the API**

#### Allowed Property Names (in general) ####

Not all kinds of parameter names are permitted, some of them even break the GunDB API:

* **Dangerous**: an attempt to write a property with an empty name `''` either breaks the node or produces strange results ( e.g., `.put({ '':'Hi' })` will actually write `{ '0':'H', '1':'i' }`);
* **Dangerous**: an attempt to write a property with the name `'_'` seems to break the node (as property `'_'` is already used by GunDB itself for a node's metadata);
* property names containing control characters (even `'\0'`) seem to work fine;
* there seems to be no explicit limit on the length of parameter names (I tried 10k which worked)

> **Conclusion: for professonal application development it will be extremely important to harden the API**

#### Allowed Property Names (when writing nested Objects) ####

Since property names play a special role when used to write nested objects (and, thus, create new GunDB nodes) they have their own restrictions:

* an attempt to write a nested object into a property with an empty name (`''`) will
  * instead write a link to the node which is currently worked on and
  * write all properties of the nested object into that node itself
* property names of the form `~`, `~@`, `~@xxx` or `~xxx` will always write their links (as intended) but fail to create the target nodes unless the constraints described [above](https://github.com/rozek/notes-on-gundb#addressing-nodes) are met

> **Conclusion: for professonal application development it will be extremely important to provide a wrapper around the original API which throws exceptions instead of logging useless messages which do not even contain any tracebacks**

### Patching Nodes ###

Multiple properties of any kind may be written in a single operation.

```
  Gun.get('a/node').put({
    'null':    null,
    'boolean': true,
    'number':  1.23,
    'string':  'Hi',
    'object':  { 'null':null, 'boolean':false, 'number':1.23, 'string':'Hi', 'object':{} },
    'link':    Gun.get('other/node'),
  })
```

The contents of the given argument are merged with the already existing contents of the addressed node: i.e., missing node properties will be created, existing ones overwritten and all other node properties left untouched.

### Waiting for Acknowledgements ###

## Observing Nodes ##

## SEA - Security, Encryption and Authorization ##

### Keypair Generation ###

### Symmetric Encryption ###

### Signing ###

### Proof-of-Work ###

### Asymmetric Encryption ###

### Certificates ###

## User Handling ##

### Creating a new User ###

### Logging in ###

### Logging out ###

### Creating a new User with an Alias ###

### Changing a User's Password ###

### When a User's Credentials get compromised.. ###

## Persistence ##

## Extending the GunDB API ##

GunDB provides an official mechanism to extend its API: by adding properties to `GUN.chain` (where `GUN` is the constructor function, not a GunDB instance) new functions can be injected which can then be called like any other method of a GunDB context:

```
  GUN.chain.Data = async function () {
    return new Promise((resolve,reject) => {
      this.once((Contents) => {
        const Result = {...Contents} // "Contents" must not be modified!
          delete Result._
        resolve(Result)
      })
    })
  }

  const Data = await Gun.get('an/existing/node').Data()
```

## Potential Extensions of the GunDB API ##

Here are the methods the author has added to GunDB

### GUN.ValueIsPlain ###

Returns `true` if the given value can be used as plain data property of a node - or `false` otherwise.

```
  GUN.ValueIsPlain = GUN.chain.ValueIsPlain = function (Value) {
    switch (typeof Value) {
      case 'boolean':
      case 'number':
      case 'string':  return true
      default:        return false
    }
  }
```

Usage:

```
  if (GUN.ValueIsPlain(Value)) { ... }
```

> Nota bene: conceptionally, this function is a _static_ method of the "class" `GUN` - but for the sake of simplicity (and to reduce the number of potential typos) it may also be applied to a GunDB _context_

### GUN.ValueIsLink ###

Returns `true` if the given value looks like a link to a GunDB node - or `false` otherwise.

```
  GUN.ValueIsLink = GUN.chain.ValueIsLink = function (Value) {
    return (
      (Value != null) && (typeof Value === 'object') &&
      (Value._ != null) && (typeof Value['#'] === 'string')
    )
  }
```

Usage:

```
  if (GUN.ValueIsLink(Candidate)) { ... }
```

> Nota bene: conceptionally, this function is a _static_ method of the "class" `GUN` - but for the sake of simplicity (and to reduce the number of potential typos) it may also be applied to a GunDB _context_

### GUN.ValueIsGarbage ###

Returns `true` if the given value is `null` - or `false` otherwise.

```
  GUN.ValueIsGarbage = GUN.chain.ValueIsGarbage = function (Value) {
    return (Value === null)
  }
```

Usage:

```
  if (GUN.ValueIsGarbage(Candidate)) { ... }
```

> Nota bene: conceptionally, this function is a _static_ method of the "class" `GUN` - but for the sake of simplicity (and to reduce the number of potential typos) it may also be applied to a GunDB _context_

### GUN.TargetIdOfLink ###

Returns the id of the node a given link is pointing to - or throws an error if the given value is not a GunDB link.

```
  GUN.TargetIdOfLink = GUN.chain.TargetIdOfLink = function (Value) {
    if (GUN.ValueIsLink(Value)) {                     // keep implementation DRY
      return Value._['#']
    } else {
      throw new TypeError('the given argument is not a GunDB link')
    }
  }
```

Usage:

```
// 'node' shall contain a property 'nextNode' which is a link
  let NodeData = await Gun.get('node').Data()
  let TargetId = GUN.TargetIdOfLink(NodeData).nextNode
```

> Nota bene: conceptionally, this function is a _static_ method of the "class" `GUN` - but for the sake of simplicity (and to reduce the number of potential typos) it may also be applied to a GunDB _context_

### GUN.TargetOfLink ###

Returns the a context of the node a given link is pointing to - or throws an error if the given value is not a GunDB link.

```
  GUN.TargetOfLink = GUN.chain.TargetOfLink = function (Value) {
    return GUN().get(GUN.TargetIdOfLink(Value))       // keep implementation DRY
  }
```

Usage:

```
// 'node' shall contain a property 'nextNode' which is a link
  let NodeData = await Gun.get('node').Data()
  let Target   = GUN.TargetOfLink(NodeData).nextNode
```

> Nota bene: conceptionally, this function is a _static_ method of the "class" `GUN` - but for the sake of simplicity (and to reduce the number of potential typos) it may also be applied to a GunDB _context_

### &lt;context&gt;.Contents ###

Retrieves the full contents (i.e., data and metadata) of a node given by its context

```
  GUN.chain.Contents = async function () {
    return new Promise((resolve,reject) => {
      this.once((Contents) => {
        return {...Contents} // "Contents" must not be modified!
      })
    })
  }
```

Usage:

```
  const Contents = await Gun.get('an/existing/node').Contents()
```

Upon completion

* `Contents._` will contain the metadata for the given node,
* `Contents._.#` its "soul", and
* `Contents._.>` the timestamps of all actual data properties.

All other properties of `Content` will contain the currently known values and links of the node.

### &lt;context&gt;.Data ###

Retrieves the payload of a node given by its context

```
  GUN.chain.Data = async function () {
    return new Promise((resolve,reject) => {
      this.once((Contents) => {
        const Result = {...Contents} // "Contents" must not be modified!
          delete Result._
        resolve(Result)
      })
    })
  }
```

Usage:

```
  const Data = await Gun.get('an/existing/node').Data()
```

### &lt;context&gt;.Metadata ###

Retrieves the metadata of a node given by its context

```
  GUN.chain.Metadata = async function () {
    return new Promise((resolve,reject) => {
      this.once((Contents) => {
        return {...Contents._} // "Contents" must not be modified!
      })
    })
  }
```

Usage:

```
  const Metadata = await Gun.get('an/existing/node').Metadata()
```

### GUN.waitFor ###

Waits for the given number of milliseconds.

```
  GUN.waitFor = GUN.chain.waitFor = async function (Duration) {
    return new Promise((resolve,reject) => {
      setTimeout(resolve,Duration)
    })
  }
```

Usage:

```
  await Gun.waitFor(100) // waits for 100ms
```

> Nota bene: conceptionally, this function is a _static_ method of the "class" `GUN` - but for the sake of simplicity (and to reduce the number of potential typos) it may also be applied to a GunDB _context_

----

(old stuff below)


### Properties and their values ###

GunDB nodes look like "associative arrays" (or "hash tables" or simply plain JavaScript objects) and may contain "properties" with "values". The values of a property may be

* `null`,
* a primitive `boolean`, `number` or `string` value or
* a "link", i.e., an object of the form `{'#':'...'}` where the ellipsis stands for the "soul" of a node (see section [Souls](https://github.com/rozek/notes-on-gundb/blob/main/README.md#souls))

Links do not necessarily have to refer to _other_ nodes - it is well permitted to directly or indirectly refer to the same node again ("circular reference")

### Souls ###

Every node has a unique global id called "soul". Nodes that belong to a user have souls looking like "~34Pi...neEo/sharedData/object" (where the ellipsis stands for a lot of additional characters, see section [User Handling](https://github.com/rozek/notes-on-gundb/blob/main/README.md#user-handling)), others "outer/inner/object". 

### Paths and Keys ###

Souls consist of a (potentially empty) "path" followed by a slash (`/`) and a "key" (e.g., "outer/inner/object") - similar to the path names of files in a file system.

Often, the various subpaths (e.g., "outer", "outer/inner") identify nodes which link to other nodes whose ids start with the same path (e.g., "outer/...", "outer/inner/...") forming a "containment tree" with "outer nodes" (those with shorter ids) "containing" "inner nodes" (those with longer ids).

But, from a technical viewpoint, ids are just plain strings and there is no need for "outer" nodes to exist when "inner" nodes do (e.g., node "outer" may be missing even when "outer/inner" exists). As a consequence, there is also no need for "outer" nodes to link to "inner" nodes for them to exist - this is completely different from file systems where "folders" have to exist or their inner files are missing as well.

Additionally, it is perfectly possible for the "key" part of a "soul" to contain a slash - the only requirement is that a the non-empty "path" in front of a "key" is followed by a slash in order to build a complete "soul". 

## User Handling ##

"[Users](https://github.com/amark/gun/wiki/User)" in GunDB are actually represented by cryptographic key pairs, "aliases" allow users to be created, looked-up and authenticated by name. Nodes with keys of the form `'~' + publicKey` on the outermost level build the foundation of "user spaces". Trying to write into the space of a given user requires a GunDB client to be authenticated as that user (which requires the proper _private_ key for the public one that builds the user space)

### Logging out ###

[user.leave](https://github.com/amark/gun/wiki/User#userleave) logs a user out - here is how you can wait for success:

```
  await (async () => {
    Gun.user().leave()
    while (Gun.user()._.sea != null) {
      await waitFor(1)
    }
  })()
```




## License ##

[MIT License](LICENSE.md)
