## HP End to End Encryption

### This repo
~~~plain
git clone https://github.com/sb-enfocus/hp-v2-end-to-end-encryption
cd hp-v2-end-to-end-encryption
npm install
~~~


This library will use [Typescript](https://www.typescriptlang.org/) since that is what Ionic v3 uses, and it will make integration with the app simpler. If you change files in the lib, you may run
~~~plain
tsc index.ts
~~~
to generate the index.js. Make sure you have tsc installed though
~~~plain
npm install -g typescript
~~~


### Docs:
#### Intro
From a broad view, here's how E2E encryption will work (within HP, and according to Virgil's API).
* Any user with the app downloaded will be assumed to have a private key that exists only on their device
  * If they do not have a private key when the app opens, the app creates one and saves it to localstorage
* Every user will also have a public key, that is stored on Virgil's "Card Server".

Every time a user creates a private key, he then creates a public key and saves on Virgil's Server. Here is how a private and public key is generated:
```typescript
import { virgil } from 'virgil-sdk';

// Create private key:
let api = virgil.API("[<VIRGIL APLICATION ID>]");
let privateKey = api.keys.generate();
// Now create a public key.
// NOTE: this will occurr on the cloud code for us,
var publicKey = api.cards.create("username", privateKey);
// Get signature on key via cloud code
// ...

// Now save the card using virigl's cloud service
api.cards.publish(publicKey)
.then(() => {

});
```


> **Taken from Virgil's documentation**
> * Virgil Card – Virgil Сards carry the user’s public key. Virgil cards are published to Virgil’s Cards Service (imagine this service is like a telephone book) for other users to retrieve them: Alice needs to retrieve Bob’s public key in order to encrypt a message for Bob using that key.
> * Virgil Key -  this is what we call a user’s private key. Remember, private keys can decrypt data that was encrypted using the matching public key.
> * npm installed on your command line


**Encrpyting**
Alice wants to send Bob a message. She uses his public key to encrypt the message (this may occur on her device or on the server) and sends the resulting hash

To get a user's public card and encrypt
```typescript
api.cards.find("bob")
.then((bobsCard) => {
    // do something with Alice's Virgil Cards
    var message = "Hey Bob, how's it going?";
    // encrypt the message
    var ciphertext = api.encryptFor(message, bobCards).toString("base64");
});
```
**Decrypting**
Bob receives the encrypted message and uses his private key to


**Saving a private key**
Virgil has built in methods for this, but documentation is hard to find for javascript / typescript. At the least, the following function works to import / export private keys such that they may be saved to the device
```typescript
let privateKey = api.keys.generate();
let exportedKey = privateKey.export();

localstorageService.save("pkey", exportedKey);

// Later session
let privateKey = api.keys.import(localstorageService.retrieve("pkey"))
// Save privateKey.export()
```
#### Group Encryption
NOTE: used the following [source](https://crypto.stackexchange.com/questions/12417/multi-party-encryption-algorithm)

Take a group of users: John, Alice, and Bob
* Each has a private key, or Virgil Key stored on their devices
* Each has a public key that others may use to encrypt messages meant for the corresponding user

Think of a group as a user. Upon creation, it will also generate a private / public key. However, it's private key will be sent to all of the user's in the group via their public keys.

So take the following scenario. An admin named Mary wants to create a group and add Chris and Sean (and herself).

* She submits the Community on the app
* The cloud code creates the Community
  * It creates a public / private key for the community
* the private key is encrypted with Chris, Sean and Mary's public key's and added to a Parse table so that they can access it.
