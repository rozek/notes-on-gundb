# notes-on-gundb #

a (growing) collection of notes on GunDB

![Network of Pearls](img/network-of-pearls.png)
<br><span style="font-size:10px">(image made with [DiffusionBee](https://github.com/divamgupta/diffusionbee-stable-diffusion-ui) and [Realistic Vision v2.0](https://civitai.com/models/4201/realistic-vision-v20))</span>

[GunDB](https://github.com/amark/gun) is a decentralized (peer-to-peer) distributed [graph database](https://en.wikipedia.org/wiki/Graph_database) with support for multiple users, access control and persistence. It runs in the browser or on [Node.js](https://nodejs.org/en), works fine when offline and synchronizes with other peers using a "last-write-wins" strategy when online.

While GunDB itself may be brilliant in many aspects, the docs are a complete nightmare (hard to read, chaotic, incomplete, wrong). This repository therefore contains a collection of the author's notes on GunDB written while he tries to learn and use it.

In the end, these notes will hopefully be less chaotic than the official docs or the GunDB wiki.

> feel free to [open an issue](https://github.com/rozek/notes-on-gundb/issues) if you find errors in my descriptions

## GunDB Documentation and Chat ##

**Do not use `https://gun.eco/docs` - use the [GunDB wiki](https://github.com/amark/gun/wiki) instead, it's much better**. The descriptions in this collection will therefore refer to the wiki rather than to the "official" docs.

If you have questions and don't find any answers in the docs, the wiki or these notes, you may try joining the [chat on GunDB](https://app.gitter.im/#/room/#amark_gun:gitter.im). 

## Basic Concepts and Terms ##

GunDB is a decentralized (peer-to-peer) distributed graph database.

The entries in this database are called **nodes**. Every node has a globally unique **id** (called its **soul**) and may contain an arbitrary number of **properties**. Properties may contain **values** (either `null` or a boolean, number or string) or **links**. Links are pointers to nodes (identified by their soul) and may either point to _other_ nodes or (directly or indirectly) to themselves: <u>circular references are deliberately permitted!</u>

At the beginning, the only known node is the **root node**. By (recursively) following the links in this node (or directly navigating to a given soul) other nodes can be visited and their properties loaded. The set of all nodes is sometimes also called the **universe** of a graph database.

Nodes directly "above" the root node play a special role, within these notes they are therefore called **trunk nodes** and all remaining nodes **branch nodes**.

In GunDB, **users** are represented by _cryptographic key pairs_. The <u>public</u> key of such a pair is used to identify a user within the GunDB universe and to provide a trunc node for her/him. The <u>private</u> key of that pair is required in order to get the permission to write into the trunc node and above.

As indicated by their name, _private_ keys should be kept safe and not shared: when lost, the related user space can no longer be written to - when leaked, everybody with that key may modify the information in the related user space. On the other side, _public_ keys may deliberately be shared - they allow others to navigate to the related user space, verify a user's signatures or encrypt messages for that user.

Since cryptographic keys look like long random numbers, users may also be referenced by a (human readable) **alias**. When created, the trunc node for such an alias contain the public key for that user and can be used to navigate to the actual user space.

It should be noted that _everybody may define new users_ for GunDB just by creating a cryptographic key pair (and the corresponding trunc node) or by creating a new alias. Furthermore, "users" do not necessarily have to represent human beings - they could also represent user _groups_, data spaces for applications, chat rooms and much more. 

The participants of a graph database are called **peers**. At the moment, GunDB provides JavaScript _clients_ for such peers, these may be used by (human-controlled) applications, relays or other automated systems


(more to come)





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

### Contexts ###

GunDB uses "contexts" (other people might prefer the term "handles") to refer to a node with a given soul






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



## Utility Functions ###

Here are the various utility functions which were used in the code snippets shown above:

### waitFor ###

```
/**** wait a given number of milliseconds ****/

  async function waitFor (Duration) {
    return new Promise((resolve,reject) => {
      setTimeout(resolve,Duration)
    })
  }
```


## License ##

[MIT License](LICENSE.md)
