# notes-on-gundb #

a (growing) collection of notes on GunDB

![Network of Pearls](img/network-of-pearls.png)

[GunDB](https://github.com/amark/gun) is a decentralized (peer-to-peer) distributed [graph database](https://en.wikipedia.org/wiki/Graph_database) with support for multiple users, access control and persistence. It runs in the browser or on [Node.js](https://nodejs.org/en), works fine when offline and synchronizes with other peers using a "last-write-wins" strategy when online.

While GunDB might be brilliant in many aspects, the docs are a complete nightmare (hard to read, chaotic, incomplete, wrong). This repository therefore contains a collection of the author's notes on GunDB written while he tries to learn and use it.

In the end, these notes will hopefully be less chaotic than the official docs or the GunDB wiki.

> feel free to [open an issue](https://github.com/rozek/notes-on-gundb/issues) if you find errors in my descriptions

## GunDB Documentation ##

**Do not use `https://gun.eco/docs` - use the [GunDB wiki](https://github.com/amark/gun/wiki) instead, it's much better**. The descriptions in this collection will therefore refer to the wiki rather than to the "official" docs.

If you have questions and don't find any answers in the docs, the wiki or these notes, you may try joining the [chat on GunDB](https://app.gitter.im/#/room/#amark_gun:gitter.im). 

## Basic Concepts and Terms ##

### Nodes and Links ###

The individual items stored in GunDB are called "nodes" and may contain plain data and/or "links" to other nodes.

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
