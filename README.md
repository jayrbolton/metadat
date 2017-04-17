# dat + pgp for encrypted, p2p user interaction and data sharing

This integrates dat with Open PGP to allow for a 100% p2p and encrypted data sharing system and user-level controls, like controlling who can see what data. This is intended to be a node API on which we could build a data-sharing UI with permissions and user interaction.

_what it does_
- privately share specific dats with specific people and groups over p2p using PGP
- send encrypted communication with your contacts over dat
- publicly share dats

# api

## setup(options, cb)

Create a new metadat user. The `options` takes these parameters:
* path: path you want to use to save your data (eg `~/.metadat`)
* name: name to use for this user
* passphrase: passphrase for generating PGP keys

`setup` takes an optional callback, which will receive the user object:

```js
const metadat = require('metadat')

metadat.setup({
  path: '~/.metadat'
, name: 'bob ross'
, passphrase: '123abc'
}, function(user) {
  // user is an object with pgp keys, dat keys, and more
})
```

## load(path, passphrase)

This will load an existing metadat root directory and unencrypt things using the given passphrase.

```js
metadat.load('~/.metadat', '123abc', function(user) {
  // the user object is provided, needed for many functions below
  user.pubKey // pgp public key
  user.publicMetadat // key of the user's public metadat
  user.relationships // other users that they have contact entries for
  user.follows // other users that they are following
})
```

## share(userA, datKey, userB, cb)

Share a regular dat with a contact. It will find the relationship metadat for this contact and add the given dat key into the list of shared dats.

```js
// User with key userA shares dat with key 'datKey' with contact  userB
metadat.share(userA, 'datKey', userB, function() {
  // Share is complete
})
```

## groupShare(userA, datKey, groupName, cb)

Share a dat with a group

```js
metadat.shareGroup(userA, 'datKey', 'groupName', function() {
  // Share is complete
})
```

## unshare(userKey, datKey, contactKey, cb)

Removes a dat key from a relationship metadat and generates a new dat key, so the contact can no longer read any updates (but will likely still have a copy on their own device).

```js
// User userA unshares dat 'datKey' with contact userB
metadat.unshare(userA, 'datKey', userB, function() {
  // Unshare is complete
})
```

## follow

Follow another user's public metadat

```js
metadat.follow(userA, userB, function(dats) {
  // dats is an array of public dats from userB
})
```

## createRelationship(userA, userB, cb)

Create a relationship metadat between two users. This creates a new dat for the relationship metadat, encrypts the key using `userB`'s pubkey, and places it in the first user's public metadat for the second user to read.

The callback is called when the relationship handshake is initiated and the encrypted relationship metadat key has been created.

```js
metadat.addContact(userA, userB, function(data) {
  data.key // relationship metadat key 
  data.keyArmored // pgp-encrypted metadat key
})
```

## addToGroup(userA, groupName, userB, cb) 

Add a contact to a group.

```js
metadat.addToGroup(userA, 'groupName', userB, function() {
  // Add to group is complete
})
```

## create(userA, path, cb)

Create a new dat, private by default. Pass in the user object and a path for the dat.

```js
metadat.create(userA, path, function(key) {
  // dat is created, dat key is now available
})
```

## publicize(userA, datKey, cb)

Make a dat public for userA. This will add it into the public listing in their public metadat.

```js
metadat.publicize(userA, 'datKey', function() {
  // dat is now public
})
```

## privatize(userA, datKey, cb)

Remove a dat key from your public listing and change the key.

```js
metadat.privatize(userA, 'datKey', function() {
  // dat is no longer public
})
```

## postMessage(userA, message, userB, cb)

Post a message from one user to another

```js
metadat.postMessage(userA, 'message content', userB, function() {
  // Message has been posted
})
```

## readMessages(userA, cb)

Read all messages for a user

```js
metadat.readMessages(userA, function(messages) {
  // messages is an array of message objects
})
```
