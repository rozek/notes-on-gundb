# notes-on-gundb #

a (growing) collection of notes on GunDB

![Network of Pearls](img/network-of-pearls.png)
<br><span style="font-size:10px">(image made with [DiffusionBee](https://github.com/divamgupta/diffusionbee-stable-diffusion-ui) and [Realistic Vision v2.0](https://civitai.com/models/4201/realistic-vision-v20))</span>

[GunDB](https://github.com/amark/gun) is a decentralized (peer-to-peer) distributed [graph database](https://en.wikipedia.org/wiki/Graph_database) with support for multiple users, access control and persistence. It runs in the browser or on [Node.js](https://nodejs.org/en), works fine when offline and synchronizes with other peers using a "last-write-wins" strategy when online.

> **Important: after two weeks of intensive work and no substantial outcome, I have decided to give up on GunDB - it is full of design flaws, bugs and - even worse - race conditions and the implementation looks like being hacked in a style used 40 years ago (when source code had to be compact and variable names short and objects to be returned by reference because of performance constraints)**
> 
> **I wish everbody working with and on GunDB good luck - but will no longer participate myself**
>
> **Nevertheless, you may still use my contributions in any way you like - they are MIT licensed**

While the goals and promised features of GunDB may sound really promising, the docs are a complete nightmare (hard to read, chaotic, incomplete, contradictory, wrong). This repository therefore contains a collection of the author's notes on GunDB written (from the viewpoint of an application developer) while he tries to learn and use it.

In the end, these notes will hopefully be more systematic than the official docs or the GunDB wiki.

> feel free to [open an issue](https://github.com/rozek/notes-on-gundb/issues) if you find errors in my notes

## Table of Contents ##

Just as a hint: you can see an outline of these notes and directly navigate to a specific section by clicking on the "list" icon left of the "README.md" title above the "notes-on-gundb" header - or at the top of this window in case that you already had to scroll down!

## Keeping up with these Notes ##

These notes are being worked on very intensively at the moment and existing text is changed or new text added in a chaotic manner whenever time permits. If you already read an earlier version of these notes and would simply like to see what changed, the following approach is recommended:

* fork this repository and read the forked version
* whenever you come back later and want to see if there are any changes and _what_ has changed, do the following:
  * GitHub will tell you that your fork is several commits behind my original (you'll find that message above the list of files in your repository)
  * open the [original notes](https://github.com/rozek/notes-on-gundb) in a separate window or tab - here you will actually read any new sections
  * back in _your_ repository, click on the link that says "n commits behind" - this will open a list of all changed files
  * locate file "README.md" to see the actual changes (called "diff"): the left side will show you which text has been removed, the right side what has been added
  * either read the changed parts directly in the "diff" itself or
  * in the original notes: use your browser's "find" command to locate any added text (of which you practically copied unique parts into the clipboard from the diff before) and read it there

This procedure should save you a lot of time while still keeping you up-to-date...

## GunDB Documentation and Chat ##

**Do not use `https://gun.eco/docs` - use the [GunDB wiki](https://github.com/amark/gun/wiki) instead, it's much better** (albeit even more chaotic). The descriptions in this collection will therefore refer to the wiki rather than to the "official" docs.

If you have questions and don't find any answers in the docs, the wiki or these notes, you may try joining the [chat on GunDB](https://app.gitter.im/#/room/#amark_gun:gitter.im). 

## Basic Concepts and Terms ##

GunDB is a decentralized (peer-to-peer) distributed graph database.

### Database Structure ###

The entries in this database are called **nodes**. Every node has a globally unique **id** (called its **soul**) and may contain an arbitrary number of **properties**. Properties may contain **values** (either `null` or a boolean, number or string) or **links**. Links are pointers to nodes (identified by their soul) and may either point to _other_ nodes or (directly or indirectly) to themselves: <u>circular references are deliberately permitted!</u>

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

Nevertheless, a network outage may always disconnect some peers from the rest of a GunDB database - this is called **network partitioning**. While GunDB in itself remains usable, remote data may not be requested, remote changes not observed and local changes not propagated to other peers - the so called **split brain problem**. However, as soon as the network becomes operational again, all changes may be delivered and the nodes held by individual peers **synchronized** with the rest of the database. As long as the delayed reported changes do not affect the same properties of the same nodes, synchronization remains simple - in case of a **conflict**, however, GunDB uses some **metadata** for the affected properties stored along in the same nodes to implement a **last-write-wins** strategy, where later changes overwrite former ones (see also the WIki pages on [Conflict Resolution](https://github.com/amark/gun/wiki/Conflict-Resolution-with-Guns), [CRDT](https://github.com/amark/gun/wiki/CRDT) and [HAM](https://github.com/amark/gun/wiki/Hypothetical-Amnesia-Machine-(HAM))).

### Eventual Consistence ###

Principally, the clients of a graph database never know if the data they received so far is complete and up-to-date - there could always remain one last peer which stores the final bit of information to make the image complete, but which has unfortunately been disconnected from the rest of the network until now.

By design, graph databases can only guarantee that their contents become **eventually consistent**, i.e., that they synchronize as soon as disconnected peers get connected again. On the other side, however, this restriction allows any client to keep working with the database even in the case of a "failure" (e.g., a network partition) as there is no central server which could be "down".

As a consequence, the clients of a graph database should always listen to (and react on) incoming changes of the nodes they have requested before. Graph databases are **dynamic databases**.

### Data Removal ###

By design, nodes and their contents cannot easily be deleted - there could always be another node that did not receive the deletion report but still holds a copy of the outdated node and sends it back to the database again. Instead, old nodes and properties have to be **nulled out** in order to be marked as **garbage**. This operation creates a **tombstone** which is kept in the database and eventually overwrites the original data in every peer. If it becomes necessary to actually get rid of no longer wanted nodes, some kind of **distributed garbage collection** has to be implemented on top of the graph database - GunDB does not (yet) provide such a DGC (see [Wiki](https://github.com/amark/gun/wiki/Delete)).

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

A graph database contains nodes and links. Each node is individually addressable and has to be fetched separately - this should be taken into account when designing the mapping between business data and GunDB nodes. Items of business objects which always belong to these objects should probably be better kept within "their" object (unless size constraints enforce a separate node). On the other hand, items used in multiple places of a business application can naturally be stored in an own node (see [Wiki](https://github.com/amark/gun/wiki/Design-Examples) for an example).

## Preparational Steps ##

GunDB may be used in a browser, under Node.js and within [NW.js](https://nwjs.io/) or [Electron.js](https://www.electronjs.org/)

### Running GunDB in a Browser ###

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

### Running GunDB under Node.js ###

(t.b.w, require('gun') vs. require('gun/gun'), automatic persistence with `file` option specifying a folder(!))

### Running GunDB under NW.js ###

(t.b.w))

### Running GunDB under Electron.js ###

(t.b.w))

### Running GunDB with Options ###

(t.b.w, see [Wiki](https://github.com/amark/gun/wiki/API-(v0.3.x)))

### Changing GunDB Options at Runtime ###

(t.b.w)

### Managing GunDB Peers ###

(t.b.w, initially and at runtime)

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
    get: ...,    // if present, contains the "key" of this context
    put: ...,    // if present, contains the known contents of the...
                 // ...represented node including any metadata
  }
}
```

When concatenating method calls in a **chain**, some methods depend on the context returned from the previous call, some don't - the [Wiki](https://github.com/amark/gun/wiki/Chaining-(v0.3.x)) contains a table which shows the individual dependencies.

## Paths and Keys ##

In a chain of `get` calls, the last `get` specifies the "context key" while all previous ones together form the "context path". On one hand, this distinction is made for internal reasons, on the other hand because "keys" have both some restrictions concerning their value but also some potential as they may address multiple nodes at once. Thus, in

`Gun.get(path_1).get(path_2).get(key)`

`path_1` and `path_2` together form the final context path while `key` contains the context key.

If there is only a single `get` call

`Gun.get(key)`

the final context's path is empty (`''`).

### Path Concatenation ###

When chaining `get` calls, GunDB concatenates the individually given arguments with a slash (`/`) in between to build the id of the final node. The outcome looks not unlike the path names used in file systems and gives the illusion(!) of a "containment tree". However, it is important to understand that there is no such tree: every node is independent of any other - and even nodes with an id like `a/b/c` linking to other nodes with ids of the form `a/b/c/d` (or similar) do not _contain_ the nodes they link to!

In order to prevent unpleasant surprises, the argument of the final `get` call (i.e., the one defining the context key) should avoid slashes, while all others may very well contain them - unless there is only a single `get`. See the following example for illustration:

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

The first three variants of `get` finally address the same node, the last one doesn't (`waitFor` is explained [below](https://github.com/rozek/notes-on-gundb#waitfor))

### Paths of nested Objects ###

To drive the illusion of a containment tree further, operations writing nested objects to a given node automatically create new nodes for the nested objects with souls built from the concatenation of the original node's id and the name of the property with the nested object:

```
  const Context_1 = Gun.get('TestObject')
    Context_1.put({ nested: {message:'hi!'} })    // writes a nested object
  console.log(await PayloadOf(Context_1)) // displays an object with a link

  const Context_2 = Gun.get('TestObject/nested') // was built automatically
  console.log(await PayloadOf(Context_2))     // displays "{message:'hi!'}"
```

(`PayloadOf` is explained [below](https://github.com/rozek/notes-on-gundb#PayloadOf))

### Taking Advantage of Paths ###

While the ids of individual nodes are as independent as the nodes themselves, common path prefixes may still be used to indicate that nodes with these prefixes semantically belong together. Particularly, such "groups" of nodes may be fetched from GunDB in a single method invocation - and, if there are many of them, the results may even be "paginated" (see [below](https://github.com/rozek/notes-on-gundb#fetching-multiple-nodes-at-once))

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
* node ids starting with `~` or `~@` may have a special semantic which becomes relevent when trying to create or write a node with such an id
* there does not seem to be an explicit limit for node ids (I tried 10k long ids which worked)

> **Conclusion: for professional application development it will be extremely important to provide a wrapper around the original API which throws exceptions instead of logging useless messages which do not even contain any stack traces**

### Addressing multiple Nodes at once ###

The previous section describes how GunDB nodes can be addressed using `get` with a literal path or key. However, invoking `get` with such a string is just the short variant of the general form

`context.get({'#':path, '.':key_pattern})`

where `path` is the context path of all nodes which shall be addressed and `key_pattern` specifies any constraints the context key should meet in order for the node to be addressed. `key_pattern` is an object with the following properties:

* `'='` specifies the exact key of a node
* `'*'` specifies the first few characters of any node's key (called the "prefix")
* `'>'` specifies the lexically lowest allowed node key (warning: behaves like "≧")
* `'<'` specifies the lexically highest allowed node key (warning: behaves like "≦")
* `'%'` specifies how many bytes a peer may load from storage before completing its current response
* `'-'` specifies whether the set of available ids should be scanned upwards or downwards?

A few examples should help understanding these properties and their semantics:

* `context.get({'#':path, '.':{ '=':key }})`<br>is just the long form of `context.get(path).get(key)`
* (t.b.w, the author could not get it to work yet)

A [Wiki page](https://github.com/amark/gun/wiki/Strings,-Keys-and-Lex-Rules) describes how the comparison works in detail.

(t.b.w, see section on LEX in the wiki page about [RAD](https://github.com/amark/gun/wiki/RAD) , Pagination)

### Reading Nodes ###

`once` can be used to retrieve (a snapshot of) the contents of a given node as a JavaScript object - including its actual payload and some metadata.

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

The "business properties" of a node may contain either

* `null`,
* a `boolean`, `number` or `string` primitive(!) or
* an object of the form `{ '#':'an-absolute-node-id' }` (called a "link")

No other property values are permitted.

> Nota bene: if preferred, you may also extend GunDB with a method which returns the actual payload of a node given by its context, as shown [below](https://github.com/rozek/notes-on-gundb#contextdata)

### Following Links ###

If a property of a given node looks like a link (i.e., it is an object with the single property `#` which contains a string), then the (always absolute) id of the link target may be extracted and used as an argument for `Gun.get()` in order to address the target node.

```
  let NodeData = await Gun.get('a/node').Data()
  let TargetNode = Gun.TargetOfLink(NodeData.link)
// assuming that 'a/node' has a property 'link' which is a node
```

> nota bene: these lines use the author's GunDB extensions [Data](https://github.com/rozek/notes-on-gundb#contextdata) and [TargetOfLink](https://github.com/rozek/notes-on-gundb#guntargetoflink)

### Writing Nodes ###

As soon as a node has been addressed using `get`, it may be created or modified using `put`.

If a given node does not yet exist, it will be created on-the-fly:

```
// assume that 'a/missing/node' does not yet exist
  Gun.get('a/missing/node').put({ 'property-name':'value' })
// the node will exist now
```

#### Writing individual Node Properties ####

Individual properties may be written by providing a plain JavaScript object containing a key-value pair with "key" being the name of the property and "value" the value to be written:

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

#### Patching Nodes ####

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

#### Allowed `put` Arguments (Node Values to be written) ####

`put` should generally be invoked with either `null` or a plain JavaScript object containing the node properties to be created or overwritten.

* an attempt to write a `boolean` or a `number` primitive into a node will effectively nullify that node as well
* an attempt to write a `string` primitive into a node will create an array-like node with the string's individual characters as its elements (which certainly isn't intended)
* **dangerous**: an attempt to write a GunDB context into a node seems to effectively replace the addressed node with the one represented by that argument (which is prseumably not intended as the adressed node's metadata now no longer contains its own id but the one of the other node)
* an attempt to write any other kind of non-plain Object into a node will log(!) an error message of the form "Invalid Data: x at y" and ignore that operation

> **Conclusion: for professional application development it will be extremely important to harden the API**

#### Influence of Node Ids on Node Creation and Modification ####

Whether it is allowed to create or modify a node depends on its id:

* nodes with ids that do not start with a tilde (`~`) may generally be created and modified
* writing to a node with the id `~` is also generally permitted
* an attempt to write a node with the id `~@` will log(!) an error _object_ containing the error message "Alias not same!" (rather than throwing an exception)
* an attempt to write to nodes with an id of the form `~@xxx` will also log(!) such an error _object_ unless the client has previously been authenticated using the alias following the `~@`
* attempts to write to nodes with an id of the form `~xxx` will work fine unless the character sequence following the `~` looks like a public key and the client has not been authenticated using a key pair containing that public key: in such a case an error _object_ with the error message "Unverified data." is logged(!) (rather than throwing an exception)

> **Conclusion: for professional application development it will be extremely important to provide a wrapper around the original API which throws exceptions instead of logging useless messages which do not even contain any stack traces**

#### Allowed Property Names (in general) ####

Not all kinds of property names are permitted, some of them even break the GunDB API:

* **dangerous**: an attempt to write a property with an empty name `''` either breaks the node or produces strange results ( e.g., `.put({ '':'Hi' })` will actually write `{ '0':'H', '1':'i' }`);
* **dangerous**: an attempt to write a property with the name `'_'` seems to break the node (as property `'_'` is already used by GunDB itself for a node's metadata);
* property names containing control characters (even `'\0'`) seem to work fine;
* there seems to be no explicit limit on the length of parameter names (I tried 10k which worked)

> **Conclusion: for professional application development it will be extremely important to harden the API**

#### Allowed Property Names (when writing nested Objects) ####

Since property names play a special role when used to write nested objects (and, thus, create new GunDB nodes) they have their own restrictions:

* an attempt to write a nested object into a property with an empty name (`''`) will
  * instead write a link to the node which is currently worked on and
  * write all properties of the nested object into that node itself
* property names of the form `~`, `~@`, `~@xxx` or `~xxx` will always write their links (as intended) but fail to create the target nodes unless the constraints described [above](https://github.com/rozek/notes-on-gundb#influence-of-node-ids-on-node-creation-and-modification) are met

> **Conclusion: for professional application development it will be extremely important to provide a wrapper around the original API which throws exceptions instead of logging useless messages which do not even contain any tracebacks**

#### Allowed Property Values ####

An attempt to assign a (nested) object to a property will create an additional node and write a link to that node into the property. The new node will have an id which consists of the original node's "soul", a slash (`/`) and the name of the property receiving the link:

```
  Gun.get('an/outer-node').put({ 'inner-node':{ data:'will be created' } })
// will create 'an/outer-node/inner-node' with the sole property 'data'
```

> **Dangerous**: an attempt to write an array into a property will log(!) the error message "Invalid data: Array at ..." (instead of throwing an exception). Trying to read the contents of the addressed node may lock the application (the `once` callback will never be called)

An attempt to assign an object of the form `{ '#':'...' }` (i.e., a link) to a property will principally write the given link. However,

* an attempt to write an empty link `{ '#':'' }` will instead write a link to a node with an id consisting of the id of the original node, a slash `/` and the name of the property written to

An attempt to write a GunDB context object into a node property will write a link to the addressed node instead:

```
  Gun.get('a/node').put({ link:Gun.get('other/node') })
// will write a link to 'other/node' into property 'link'
```

An attempt to write any other kind of non-plain Object into a node property will log(!) an error message of the form "Invalid data: x at y" and ignore that operation

These behaviours are independent of whether the target node exists or not.

> **Conclusion: for professional application development it will be extremely important to harden the API**

#### Waiting for Acknowledgements ####

[put](https://github.com/amark/gun/wiki/API-(v0.3.x)#put) accepts a callback as its second argument. This callback is invoked whenever the given put request is acknowledged by a peer.

> **Nota bene**: if your application runs offline or (deliberately) without peer, the callback will never be invoked - which makes that feature effectively useless

> **Conclusion: for professional application development it will be extremely important to implement an alternative which asserts that the new node contents have been persisted, at least. Ideally, there should even be a distinction between persistence and synchronization ACKs**

## Observing Nodes ##

Since a graph database is _dynamic_ by design, it is important for an application to observe any nodes it is interested in. GunDB provides three methods for that purpose:

* [on](https://github.com/amark/gun/wiki/API-(v0.3.x)#on),
* [once](https://github.com/amark/gun/wiki/API#-gunoncecallback-option) and
* [map](https://github.com/amark/gun/wiki/API-(v0.3.x)#map)

which can even be combined in various ways.

(t.b.w, `once` and `map` can be used in two ways: with callbacks or as part of a chain)

### Fetching a Node as a whole - once now ###

(t.b.w)

### Fetching a Node as a whole - now and upon changes ###

(t.b.w)

### Iterating over the Properties of a Node ###

(t.b.w)

### Combining on, once and map ###

(t.b.w)

## Built-in Data Structures ##

Apart from plain objects with properties of type `boolean`, `number`, `string` or "links", GunDB supports two more classes of data structures out-of-the-box:

* (unordered) sets and
* time graphs

Other data types will either have to mapped onto the built-in ones or implemented on top of GunDB (see below)

### Unordered Sets ###

(t.b.w, see [Wiki](https://github.com/amark/gun/wiki/API-(v0.3.x)#set))

### Time Graphs ###

(t.b.w, see [Wiki](https://github.com/amark/gun/wiki/Timegraph))

## SEA - Security, Encryption and Authorization ##

[SEA](https://github.com/amark/gun/wiki/SEA) is used to implement GunDB's [security model](https://github.com/amark/gun/wiki/Security) (see [Wiki](https://github.com/amark/gun/wiki/Security,-Authentication,-Authorization)). It has to be loaded separately

```
<script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gun/sea.js"></script>
```

and may then be used to generate cryptographic key pairs, encrypt and sign data or create [certificates](https://github.com/amark/gun/wiki/SEA.certify). SEA works asynchronously, if you want to wait for results you should therefore wrap your code in an asynchronous IIFE

```
<script>
  const SEA = GUN.SEA
  
  ;(async () => {
    ...
  })()
</script>
```

By default, SEA methods do not throw any exceptions but simply return `undefined` if they encounter any failures. If you want SEA to throw, you have to explicitly configure

```
  SEA.throw = true
```

### Keypair Generation ###

`SEA.pair` generates two new pairs of cryptographic keys

```
  let KeyPair = await SEA.pair()
  console.log('KeyPair',KeyPair)
```

It returns an object with the following properties:

```
{
  epriv: 'DArC...CW5c',             // 43 char.s
  epub:  'smJU...0jeI.wSCl...HpG0', // 87 char.s
  priv:  '"grPs...phao"',           // 43 char.s
  pub:   '9Su...ho1A.GU--...l0Wo',  // 87 char.s
}
```

`epriv` and `priv` are the private parts of a key pair and should be kept safe and not shared, `epub` and `pub` are their public counterparts and may safely be published (in fact, they often _have_ to be published)

### Symmetric Encryption ###

`SEA.encrypt` can be used to encrypt a given text, and `SEA.decrypt` to decrypt it again. Both methods require the same cipher in form of a literal passphrase but also accept a key pair generated with `SEA.pair` from which they then take the `epriv` part

```
  let originalText  = 'Lorem ipsum dolor sit amet'
  let encryptedText = await SEA.encrypt(originalText, 't0ps3cr3t!')
  let decryptedText = await SEA.decrypt(encryptedText,'t0ps3cr3t!')

  console.log('originalText ',originalText)
  console.log('encryptedText',encryptedText)
  console.log('decryptedText',decryptedText)
  console.log(originalText === decryptedText)
```

And now with a key pair

```
  let KeyPair = await SEA.pair()

  let originalText  = 'Lorem ipsum dolor sit amet'
  let encryptedText = await SEA.encrypt(originalText, KeyPair)
  let decryptedText = await SEA.decrypt(encryptedText,KeyPair)

  console.log('originalText ',originalText)
  console.log('encryptedText',encryptedText)
  console.log('decryptedText',decryptedText)
  console.log(originalText === decryptedText)
```

Encrpyted text is a string of the form

`'SEA{"ct":"YK0o...p6Lp","iv":"6vYQWNafgnDxfN6SJtQa","s":"URgUSMPvEWj0"}'`

and can directly be written into a node property.

### Signing ###

In order to verify the origin of a given message, that text can be signed with `SEA.sign` (using the private part of a key pair) and verified with `SEA.verify` (using the public part of a key pair): The receiver of a signed message must therefore know the public key of its originator

```
  let KeyPair = await SEA.pair()

  let originalText = 'Lorem ipsum dolor sit amet'
  let signedText   = await SEA.sign(originalText,KeyPair)
  let acceptedText = await SEA.verify(signedText,KeyPair.pub)

  console.log('originalText',originalText)
  console.log('signedText  ',signedText)
  console.log('acceptedText',acceptedText)

  let brokenText   = signedText.replace(/Lorem/,'lorem')
  let rejectedText = await SEA.verify(brokenText,KeyPair.pub)

  console.log('brokenText  ',brokenText)
  console.log('rejectedText',rejectedText) // undefined
```

Signed text is a string of the form

`SEA{"m":"Lorem ipsum dolor sit amet","s":"LWqj...iw=="}`

and can directly be written into a node property.

If transmitted "untampered" `SEA.verify` will extract the original message from the signed text and return it - otherwise it will simply return `undefined`

### Proof-of-Work ###

In order to put some computational burden on a client and delay certain operations (e.g., to prevent flooding a database or hamper brute force attacks on passwords), a "proof-of-work" can be requested

```
  let Text = 'Lorem ipsum dolor sit amet' // could be a password
  let Salt = Math.random()

  let Proof = await SEA.work(Text,Salt)
  console.log('Proof',Proof)
  
  console.log('same text', 
    Proof === await SEA.work('Lorem ipsum dolor sit amet',Salt) 
    ? 'matches' 
    : 'differs'
  )
  console.log('other text',
    Proof === await SEA.work('lorem ipsum dolor sit amet',Salt) 
    ? 'matches' 
    : 'differs'
  )
```

This computes a hash value for the given text using the given `Salt` value. `Salt` should be random (in order to prevent hackers from pre-computing hash values for multiple passwords) but does not have to be kept very secret.

A common use case for "proof-of-work" is the password-based authentication of users: instead of storing the passwords themselves (as they could become compromised if an attacker would get access to the password database) their proof-of-work hash should be stored (since attackers would now have to run compute-intensive brute force attacks on every single password - individually, if every password's `Salt` differs from any other). Upon login, the given password is hashed again (with the same `Salt` as before) and compared to the stored one: if equal, access is granted, otherwise rejected.

`SEA.work` has the signature `SEA.work(data, salt, callback, options)` and can be configured in multiple ways by providing an object with `options`:

```
  const options = {
    name:'PBKDF2', // or 'SHA-256'
    encode:'base64', // or 'utf8', 'hex'
    iterations:10000, // number of iterations
    salt:..., // salt value to use
    hash:..., // hash method to use
    length:... // ?
  }
```

Since the result of running `SEA.work` is a cryptographic hash (and looks like a really random number), it can also be used for other purposes where such a number should be deterministically derived from some input - but with some computational burden in order to limit the number of brute force attacks per second.

The [wiki](https://github.com/amark/gun/wiki/SEA) gives an example where a user's password is encrypted using a "proof-of-work" built from the answers of two security questions (which only the password owner should know). The encrypted password is then stored and decrypted upon request when the same security questions are correctly answered again.

### Asymmetric Encryption ###

While a "symmetric" encryption requires the same cipher to be known both on the encrypting and the decrypting side (and allows anybody with access to that cipher to encrypt and decrypt at will), "asymmetric" encryption does not: here, a message is encrypted with the sender's _private_ and the receiver's _public_ key and decrypted with the sender's _public_ and the receiver's _private_ key - effectively limiting communication to the given sender and receiver.

`SEA.secret` can be used for that purpose as it derives a (symmetric) encryption/decryption key from a given public and private key belonging to different participants

```
  let Alice = await SEA.pair()
  let Bob   = await SEA.pair()

/**** from Alice to Bob - only ****/

  let originalText  = 'Lorem ipsum dolor sit amet'
  let encryptedText = await SEA.encrypt(originalText, await SEA.secret(Bob.epub, Alice))
  let decryptedText = await SEA.decrypt(encryptedText,await SEA.secret(Alice.epub, Bob))

  console.log('originalText ',originalText)
  console.log('encryptedText',encryptedText)
  console.log('decryptedText',decryptedText)
  console.log(originalText === decryptedText)
```

If a message shall be published once for multiple users with different key pairs, an approach can be used that has formerly be introduced by [Pretty Good Privacy](https://en.wikipedia.org/wiki/Pretty_Good_Privacy):

* create an arbitrary (practically random) encryption key and (symmetrically) encrypt your message with it
* now asymmetrically encrypt the encryption key itself for every intended receiver
* then publish both the encrypted message and all encrypted encryption keys

The receivers now either have to

* decrypt the encryption key made for them personally or
* simply try to decrypt all given encryption keys until they succeed

before they can actually decrypt the message itself. The [Wiki](https://github.com/amark/gun/wiki/Snippets) contains an example for this use case.

### Certificates ###

(t.b.w, see [Wiki](https://github.com/amark/gun/wiki/SEA.certify))

## User Handling ##

(t.b.w, GunDB provides some cryptographic protection for users, users do not have to be human beings)
(see [Wiki](https://github.com/amark/gun/wiki/User))

### Public Space vs. User Space ###

(t.b.w, see [Wiki](https://github.com/amark/gun/wiki/Introduction))

### Creating a new User ###

(t.b.w, just a cryptographic key pair)

### Logging in ###

(t.b.w, Gun.user())

### Logging out ###

(t.b.w)

[user.leave](https://github.com/amark/gun/wiki/User#userleave) logs a user out - here is how you can wait for success:

```
  await (async () => {
    Gun.user().leave()
    while (Gun.user()._.sea != null) {
      await waitFor(1)
    }
  })()
```

### Creating a new User with an Alias ###

(see [Wiki](https://github.com/amark/gun/wiki/Auth))

### Access Control ###

(t.b.w, SEA certificates)

### Changing a User's Password ###

(t.b.w, Alias only)

### When a User's Credentials get compromised.. ###

(t.b.w, change password, copy to new user space)

## Implementing Data Types ##

GunDB supports only very few data types out-of-the-box, all others will either have to be mapped onto the built-in ones or implemented on top of GunDB. The following sections provide a few examples

### Tuples and Structures ###

(t.b.w)

### Flat Ordered Sets ###

(t.b.w)

### Hierarchical Ordered Sets ###

(t.b.w)

### Sharing Text with Operational Transformation ###

GunDBs conflict resolution uses a "last-write-wins" strategy - which may work well with fine-granulated data structures, but may be really frustrating when multiple users try to work on the same text. Fortunately, there are already several other conflict resolution mechanisms - and one of them (called "Operational Transformation") has been especially developed for text sharing

(t.b.w)

## Persistence ##

(t.b.w, see [Building Modules 1](https://github.com/amark/gun/wiki/Building-Modules-for-Gun-(v0.5.x)), [Building Modules 2](https://github.com/amark/gun/wiki/Building-Modules-for-Gun), [RAD](https://github.com/amark/gun/wiki/RAD) and [Radisk](https://github.com/amark/gun/wiki/Radisk) seem to be completely broken and [will be replaced soon](https://github.com/amark/gun/issues/1329#issuecomment-1556079655))
([official storage engines](https://github.com/amark/gun/wiki/Storage))

## Relay Peers ##

(t.b.w, sometimes also called "GunDB Server", see [Wiki](https://github.com/amark/gun/wiki/Local-Desktop-Gun-Relay-(Windows,-Linux,-MAC)) for setup, or [here](https://github.com/amark/gun/wiki/Server-with-basic-example-(gun-in-action)) for own development)

### Running a Relay on an Oracle "Always-free" VM ###

(t.b.w, see [Wiki](https://github.com/amark/gun/wiki/gun-relay-manual-setting-up-(cert---ssl---container---oracle-cloud)))

## GunDB and HATEOAS ##

(t.b.w, having a highly distributed real database brings the [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) concept to a new level)

## Hooking into GunDB ##

(t.b.w, also called "module" or "plugin" in the GunDB docs, see [Building Modules for Gun](https://github.com/amark/gun/wiki/Building-Modules-for-Gun), [Building Modules for Gun (v0.5.x)](https://github.com/amark/gun/wiki/Building-Modules-for-Gun-(v0.5.x)), [Building Storage Adapters](https://github.com/amark/gun/wiki/Building-Storage-Adapters))

## Extending the GunDB API ##

GunDB provides an [official mechanism](https://github.com/amark/gun/wiki/Adding-Methods-to-the-Gun-Chain) to extend its API: by adding properties to `GUN.chain` (where `GUN` is the constructor function, not a GunDB instance) new functions can be injected which can then be called like any other method of a GunDB context:

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

Here are some methods the author has added to his GunDB instances

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
/**** Contents - fetches data and metadata of a given node ****/

  GUN.chain.Contents = async function (Timeout = 100, Default = undefined) {
    return new Promise((resolve,reject) => {
      const TimerId = setTimeout(() => {
        resolve(Default)          // the requested node seems not to exist (yet)
      },Timeout)

      this.once((Contents) => {
        clearTimeout(TimerId)
        resolve({...Contents})               // "Contents" must not be modified!
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
/**** Data - fetches the data part of a given node ****/

  GUN.chain.Data = async function (Timeout = 100, Default = undefined) {
    return new Promise((resolve,reject) => {
      const TimerId = setTimeout(() => {
        resolve(Default)          // the requested node seems not to exist (yet)
      },Timeout)

      this.once((Contents) => {
        clearTimeout(TimerId)

        const Result = {...Contents}         // "Contents" must not be modified!
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
/**** Metadata - fetches the metadata part of a given node ****/

  GUN.chain.Metadata = async function (Timeout = 100, Default = undefined) {
    return new Promise((resolve,reject) => {
      const TimerId = setTimeout(() => {
        resolve(Default)          // the requested node seems not to exist (yet)
      },Timeout)

      this.once((Contents) => {
        clearTimeout(TimerId)
        resolve({...Contents._})             // "Contents" must not be modified!
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

## Utilities ##

The following methods have been used by the author during his evaluation of GunDB (note: these are not added to `GUN.chain`!)

### writeTestObject ###

```
/**** writeTestObject - writes a test object into a given node ****/

  const TestObject = {
    'null':    null,
    'boolean': true,
    'number':  1.23,
    'string':  'Hi',
    'link':    { '#':'id-of-another-node' },
    'object':  { 'null':null, 'boolean':false, 'number':1.23, 'string':'Hi', 'object':{} },
  }

  function writeTestObject (Context) {
    Context.put(TestObject)
  }
```

### writeNestedNodes ###

creates a lot of nested nodes starting from a given base node (`Base` "inner" nodes with the ids `0`...`Base-1` per "outer" node, `Depth` levels deep)

```
/**** writeNestedNodes - recursively creates nested nodes ****/

  function writeNestedNodes (Context, Base = 10, Depth = 3, BaseKey = '') {
    for (let i = 0; i < Base; i++) {
      const currentKey     = (BaseKey === '' ? '' : BaseKey + '/') + i
      const currentContext = Context.get(''+i)

      currentContext.put({ value:currentKey })

      if (Depth > 1) {
        writeNestedNodes(currentContext,Base,Depth-1,currentKey)
      }
    }
  }
```

Here is how the number of created nodes (aka `EntryCount`) can be calculated:

```
  let EntryCount = 0, Base = 10, Depth = 3
  for (let i = 1; i <= Depth; i++) {
    EntryCount += Base ** i
  }
```

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




## License ##

[MIT License](LICENSE.md)
