# notes-on-gundb #

a (growing) collection of notes on GunDB

[GunDB](https://github.com/amark/gun) is a decentralized (peer-to-peer) distributed [graph database](https://en.wikipedia.org/wiki/Graph_database) with support for multiple users, access control and persistence. It runs in the browser or on [Node.js](https://nodejs.org/en), works fine when offline and synchronizes with other peers using a "last-write-wins" strategy when online

While GunDB might be brilliant in some aspects, the docs are a complete nightmare (incomplete, wrong, spread over many separate documents). This repository therefore contains a collection of the author's notes on GunDB as he starts learning and evaluating it.

> **First of all: do not use `https://gun.eco/docs` - use the [GunDB wiki](https://github.com/amark/gun/wiki) instead, it's much better**


## Basic Concepts and Terms ##

### Soul ###

Every node has a unique global id called "soul". Nodes that belong to a user have souls looking like "~34Pi...neEo/sharedData/object" (where the ellipsis stands for additional characters, see section "User Handling"), others "sharedTest/sharedData/object". 

### Path and Key ###

Souls consist of a (potentially empty) "path" followed by a slash `/` and a "key" (e.g., "sharedTest/sharedData/object") - similar to the path names of files in a file system.

Often, the various subpaths (e.g., "sharedTest", "sharedTest/sharedData") identify nodes which link to other nodes whose ids start with the same path (e.g., "sharedTest/...", "sharedTest/sharedData/...") forming a "containment tree" with "outer nodes" (those with shorter ids) "containing" "inner nodes" (those with longer ids) - but, from a technical viewpoint, ids are just plain strings.

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
