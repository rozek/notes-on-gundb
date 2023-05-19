# notes-on-gundb #

a (growing) collection of notes on GunDB

[GunDB](https://github.com/amark/gun) is a decentralized (peer-to-peer) distributed [graph database](https://en.wikipedia.org/wiki/Graph_database) with support for multiple users, access control and persistence. It runs in the browser or on [Node.js](https://nodejs.org/en), works fine when offline and synchronizes with other peers using a "last-write-wins" strategy when online

While GunDB might be brilliant in some aspects, the docs are a complete nightmare (incomplete, wrong, spread over many separate documents). This repository therefore contains a collection of the author's notes on GunDB as he starts learning and evaluating it.

> **First of all: do not use `https://gun.eco/docs` - use the [GunDB wiki](https://github.com/amark/gun/wiki) instead, it's much better**






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
