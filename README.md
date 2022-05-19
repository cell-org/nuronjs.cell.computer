![nuronjs.png](nuronjs.png)

# nuron.js

> JavaScript library for interacting with Nuron

nuron.js is an HTTP client that talks to Nuron to create and manage NFTs without relying on 3rd parties.

Using `nuron.js` You can make HTTP requests to your local `nuron` to make it do things for you, such as:

1. add data to the file system
2. build and sign tokens
3. pin files to IPFS

---

# Install


## Browser

```
<script src="https://unpkg.com/nuronjs/dist/nuron.js"></script>
```

## Node.js

```
npm install nuronjs
```

---

# API

## core

### constructor

Nuron stores everything under the `~/__nuron__` (under your home directory) folder on your machine.

A typical folder space for your Nuron workspaces will look like this:


```
<YOUR_HOME_DIRECTORY>
│
└── __nuron__
    └── v0
        └── home
            ├── config.json
            └── workspace
                │ 
                ├── <WORKSPACE 1>
                │   ├── db
                │   │   └── mixtape.db
                │   ├── fs
                │   │   ├── <IPFS file 0>
                │   │   ├── <IPFS file 1>
                │   │   ├── <IPFS file 2>
                │   │   ├── . . .
                │   │   └── <IPFS file n>
                │   └── web
                │       ├── index.html
                │       └── token.html
                │
                └── <WORKSPACE 2>
                    ├── db
                    │   └── mixtape.db
                    ├── fs
                    │   ├── <IPFS file 0>
                    │   ├── <IPFS file 1>
                    │   ├── <IPFS file 2>
                    │   ├── . . .
                    │   └── <IPFS file n>
                    └── web
                        ├── index.html
                        └── token.html
```

#### syntax

```javascript
const nuron = new Nuron(config)
```

##### parameters

- `config`
  - `key`: the BIP44 key derivation path. For Ethereum and its testnets, you can use `"m'/44'/60'/0'/0/0"`.
  - `workspace`: the nuron workspace name. you can create as many workspaces as you want.
  - `domain`: the context in which the tokens are valid. made up of 3 attributes:
    - `address`: the NFT contract address. All tokens created with this nuron instance can only be minted to this contract address.
    - `chainId`: the chainId (which host blockchain will the NFTs be stored on?). In case of Ethereum mainnet, it's 1. In case of Rinkeby testnet, it's 4. Learn more about chainIds here: https://chainlist.org/
    - `name`: the name of the NFT contract. this MUST match the name of the Cell contract you're trying to mint to.

##### return value

- `nuron`: initialized nuron client that can talk to the local nuron engine.

#### examples

```javascript
const nuron = new Nuron({
  key: "m'/44'/60'/0'/0/0",
  workspace: "cube",
  domain: {
    "address":"0x93f4f1e0dca38dd0d35305d57c601f829ee53b51",
    "chainId":4,
    "name":"_test_"
  }
})
```

Above code declares that every request made from this instantiated client will:

1. sign with the key at key path `m'/44'/60'/0'/0/0`
2. write all the files under the `cube/fs` folder
3. write all the DB entries inside a SQLite DB at path `cube/db/mixtape.db`
3. use the specified domain for creating tokens.

It will create a folder structure that looks like this:

```
<YOUR_HOME_DIRECTORY>
│
└── __nuron__
    └── v0
        └── home
            ├── config.json
            └── workspace
                └── cube
                    ├── db
                    │   └── mixtape.db
                    ├── fs
                    └── web
                        ├── index.html
                        └── token.html
```

From this point on,

1. When you call `nuron.fs.write(data)`, Nuron will store the data under the `~/__nuron__/v0/home/workspace/cube/fs` folder (the file system)
2. When you call `nuron.db.write(table, data)`, Nuron will store the data inside the `~/__nuron__/v0/home/workspace/cube/db/mixtape.db` SQLite file.

> Note that you DO NOT have to worry about how all of this is stored when using Nuron, since you can simply use the Nuron API to interact with the DB and the File System.


### config()

#### syntax

```javascript
let config = await nuron.config()
```

##### parameters

- none

##### return value

- `config`
  - `ipfs`: the ipfs related config attribute
    - `key`: API key for the pinning service (currently nft.storage)
  - `fs`
    - `home`: the local folder path where the nuron file system is stored



#### examples

```javascript
let response = await nuron.config()
```

should return a response that looks like this:

```json
{
  "ipfs": {
    "key": <nft.storage APK key>
  },
  "fs": {
    "home": <the folder path where the nuron file system is located on the machine>
  }
}
```

### configure()

#### syntax


```javascript
let response = await nuron.configure(config)
```

##### parameters

- `config`
  - `ipfs`: the IPFS config
    - `key`: the API key for the IPFS pinning service (currently nft.storage)

##### return value

- `response`: 
  - `success`: `true` if successful
  - `error`: an error message if there was an error


#### examples

when you run the following code:

```javascript
let response = await nuron.configure({
  ipfs: {
    key: nft_storage_key
  }
})
```

if successful you will get:

```json
{
  "success": true
}
```

if failed you will get:

```json
{
  "error": <error message>
}
```

---

## wallet

<!--

### 1.2. wallets()

get the list of addresses for the chainId

#### syntax

```javascript
const wallets = await nuron.wallet.wallets(chainId)
```

##### parameters

- `chainId`: The chainId from which to request wallet addresses. (learn more about chainId here: https://chainlist.org/)

##### return value

- `chain`: blockchain symbol (example: "ETH" for Ethereum)
- `chainId`: the requested chainId
- `accounts`: an array of accounts for the currently logged in user, for the requested chainId. Each account has the following attributes:
  - `address`: The address
  - `path`: The BIP44 derivation path


#### examples

```javascript
// Get Ethereum addresses
const wallets = await nuron.wallet.wallets(1)

// Get Rinkeby addresses
const wallets = await nuron.wallet.wallets(4)
```

returns:

```json
{
  chain: 'ETH',
  chainId: 1,
  accounts: [
    {
      address: '0xFb7b2717F7a2a30B42e21CEf03Dd0fC76Ef761E9',
      path: "m/44'/60'/0'/0/0"
    },

    ...

    {
      address: '0x123A66Be1A490129757Deb9F981095342700967d',
      path: "m/44'/60'/0'/0/2"
    },
    {
      address: '0x1dc9587797d15A9a11d3324849A22d4953497C58',
      path: "m/44'/60'/0'/0/8"
    },

    ...

    {
      address: '0x753dD1E899ca3cc6cCd8271ee7C23a9379F382A0',
      path: "m/44'/60'/0'/0/99"
    }
  ]
}
```

-->


### accounts()

Get all local account names (The names you save seed phrases under:)

#### syntax

```javascript
let accounts = await nuron.wallet.accounts()
```

##### parameters

- none

##### return value

- `accounts`: An array of local Nuron account names


#### examples

```javascript
let usernames = await nuron.wallet.accounts()
```

Assuming that you have created 3 accounts named "alice", "bob", and "carol", each with its own seed phrase, the `usernames` from the code above will be:

```json
["alice", bob", "carol"]
```


### session()

get the currently connected 

#### syntax

```javascript
let session = await nuron.wallet.session()
```

##### parameters

- none

##### return value

- `session`: the currently connected session account
  - `username`: the local nuron account name
  - `address`: the address for the currently connected key path (initialized through Nuron constructor)

#### examples


```javascript
let session = await nuron.wallet.session()
```

Assuming that the currently logged in account name is "alice", and the `nuron` object was initialized with BIP44 derivation path `"m/44'/60'/0'/0/0"`, the value of `session` will be:

```json
{
  username: "alice",
  address: <Alice's derived address at path m/44'/60'/0'/0/0>
}
```

### chain()

Get the chain information for a given `chainId`, returning all the associated accounts as well.

#### syntax

```javascript
let info = await nuron.wallet.chain(chainId)
```

##### parameters

- `chainId`: the blockchain chainId (see https://chainlist.org/)

##### return value

- `info`: an object containing the following attributes:
  - `name`: the blockchain symbol as defined in https://chainlist.org/
  - `chain`: the blockchain chainId
  - `accounts`: an array of accounts, each of which consists of the following attributes:
    - `address`: the derived account address
    - `path`: the bip44 derivation path for the address

#### examples

returns:

```json
{
  "name": "alice",
  "chain": "ETH",
  "chainId": 1,
  "accounts": [{
    "address": <address 0>,
    "path": <bip44 derivation path 0>,
  }, {
    "address": <address 1>,
    "path": <bip44 derivation path 1>,
  }, {
    ...
  }, {
    "address": <address 99>,
    "path:" <bip44 derivation path 99>,
  }]
}
```


### connect()

login to an account

#### syntax

```javascript
let response = await nuron.wallet.connect(password, username)
```

##### parameters

- `password`: the decryption password for the seed phrase stored under the account
- `username`: the name of the account under which the seed phrase is stored

##### return value

- none

#### examples

### disconnect()

log out of the current nuron session

#### syntax

```javascript
let response = await nuron.wallet.disconnect()
```

##### parameters

- none

##### return value

- none

### delete()

delete the wallet (the seed phrase) of the **currently logged in session**, and log out of the session.

#### syntax

```javascript
await nuron.wallet.delete(password)
```

##### parameters

- `password`: the encryption/decryption password for the wallet (seed phrase)

##### return value

- none

#### examples

### export()

export the seed phrase of the current sessioin, or export a specific private key at a given bip44 derivation path.

#### syntax

```javascript
let response = await nuron.wallet.export(password, key)
```

##### parameters

- `password`: the encryption/decryption password for the current session's wallet
- `key`: **(optional)** the bip44 derivation path 

##### return value

- `response`: the response to the `export()` action
  - `exported`
    - if the optional `key` variable was specified during the function call, returns the private key at the specified derivation path.
    - if the optional `key` variable was NOT specified during the function call, returns the decrypted seed phrase

#### examples

##### 1. exporting a seed phrase

for example, assuming the password is correct, the following function call:

```javascript
let exported = await nuron.wallet.export("this is a password")
```

may return the full seed phrase:

```json
dog car almond peach light water ...
```

##### 2. exporting a private key

you can also export a specific private key for the seed phrase at a specific derivation path.

```javascript
let response = await nuron.wallet.export("this is a password", "m/44'/60'/0'/0/0"))
```

may return the private key derived from the seed phrase at the bip44 path `m/44'/60'/0'/0/0`:

```json
0xD4531C095ba4abfd322d3EC58D693846b09Ad719
```

### import()

import a seed phrase and create a new local account

#### syntax

```javascript
let response = await nuron.wallet.import(password, seed, username)
```

##### parameters

- `password`: the encryption password to encrypt the seed with before storing
- `seed`: the seed phrase string to import
- `username`: the local account username to save the wallet under

##### return value

returns the imported wallet:

- `imported`
  - `name`: the name of the imported account
  - `seed`: the imported seed phrase
  - `encrypted`: the encrypted seed


#### examples

```javascript
let response = await nuron.wallet.import(
  "this is the wallet encryption password",   // the encryption password
  "dog coffee speaker pinao ...",             // the seed phrase to store
  "bob"                                       // the local nuron account username
)
```

### generate()

generate a random seed phrase and create a new local account

#### syntax

```javascript
let generated = await nuron.wallet.generate(password, username)
```

##### parameters

- `password`: the encryption password to encrypt the generated seed with before storing
- `username`: the local account username to save the wallet under

##### return value

- `generated`
  - `name`: the name of the generated account
  - `seed`: the generated seed phrase
  - `encrypted`: the encrypted seed

#### examples

```javascript
let generated = await nuron.wallet.generate(
  "this is the wallet encryption password",   // the encryption password
  "bob"                                       // the local nuron account username
)
```

## token

### build()

builds an unsigned token according to the payload description:

#### syntax

```javascript
let unsignedToken = await nuron.token.build(body)
```

##### parameters

- `body`: The same `body` payload as the `description.body` argument of the [c0js build() method](https://c0js.cell.computer/#/?id=_21-build)

##### return value

- `unsignedToken`: returns the generated token (without a signature)


#### examples

##### 1. free mint

Let's create a token that can be minted by anyone for free, first come first served.

First, we assume that we already know the domain information. You can find the domain JSON for your contract at [cell.computer dashboard](https://cell.computer)

```javascript
let unsignedToken = await nuron.token.build({
  cid: cid
})
```

##### 2. priced mint

Let's create a token that requires 1ETH payment for minting:

```javascript
let unsignedToken = await nuron.token.build({
  cid: cid,
  value: 10 ** 18                                         // 10^18wei => 1ETH
})
```

##### 3. anyone in the group can mint

sometimes you want to allow anyone from a group to mint a single token, first come first served basis.

The difference with the `sender` attribute is that the `sender` directly specifies a single address that can mint, whereas `senders` is an array, and anyone from this array can mint but only one will succeed. Here's an example:

```javascript
let signedToken = await c0.token.create({
  cid: cid,
  senders: [
    address0,
    address1,
    address2
  ]
})
```

### sign()

takes an unsigned token (created with `build()`), signs it with Nuron, and returns the signed token

#### syntax

```javascript
let signedToken = await nuron.token.sign(unsignedToken)
```

##### parameters

- `unsignedToken`: a token object without a signature, created with `nuron.token.build()` or `cell.token.build()`

##### return value

- `signedToken`: a signed token. The only difference from the `unsignedToken` is that a signature has been attached to the `unsignedToken` at path `unsignedToken.body.signature`

#### examples

```javascript
let unsignedToken = await nuron.token.build({
  cid: cid
})
let signedToken = await nuron.token.sign(unsignedToken)
```

### create()

the `create()` method builds an unsigned token and signs and returns the signed token in a single function call.

#### syntax

```javascript
let signedToken = await nuron.token.create(body)
```

##### parameters

- `body`: the same `body` object as the `nuron.token.build()` method.

##### return value

- `signedToken`: a fully signed token

#### examples

We saw above that we can build an unsigned token and then sign it to get the signed token, like this:

```javascript
let unsignedToken = await nuron.token.build({
  cid: cid
})
let signedToken = await nuron.token.sign(unsignedToken)
```

With `create()` we can merge the two into one `create()` call:

```javascript
let signedToken = await nuron.token.create({
  cid: cid
})
```


## fs

The nuron file system API


### write()


#### syntax

```javascript
let cid = await nuron.fs.write(data)
```

##### parameters

- `data`: whatever data you want to save to nuron. it can be of the following types, both in the browser and in node.js:
  - `Buffer`
  - `Uint8Array`
  - `ArrayBuffer`
  - `ReadableStream`
  - `File`
  - `Blob`
  - `String`
  - `JSON`


##### return value

- `cid`: the calculated IPFS CID of the stored file

> NOTE: the IPFS CID is computed WITHOUT publishing the files to the global IPFS network.
>
> "save()" only stores the files locally to your Nuron file system.
>
> To publish publicly to the global IPFS network, you can use the pin() method, explained in the next section.

#### examples

##### 1. storing a JSON object

```javascript
// Storing JSON
let cid = await nuron.fs.write({
  name: "NFT 1",
  description: "first nft",
  image: "ipfs://bafkreidztp557q7fvbq5t34uat5l3vztkcjzavatrohclmigh5o7qxyrq4"
})
```

##### 2. storing a Buffer object

```javascript
let cid = await nuron.fs.write(Buffer.from("hello world"))
```

##### 3. storing a File object

```html
<html>
<body>
<script src="https://unpkg.com/nuronjs/dist/nuron.js"></script>
<input type='file' id='file'>
<script>
const nuron = new Nuron({
  key: "m'/44'/60'/0'/0/0",
  fs: "filetoken",
  domain: {
    "address":"0x93f4f1e0dca38dd0d35305d57c601f829ee53b51",
    "chainId":4,
    "name":"_test_"
  }
})
document.querySelector("#file").addEventListener("change", async (e) => {
  let cid = await nuron.fs.write(e.target.files[0])
  alert("saved " + cid)
})
</script>
</body>
</html>
```

### rm()

remove cids from the file system

#### syntax

```javascript
await nuron.fs.rm(cids)
```

##### parameters

- `cids`
  - **cid array:** An array of IPFS CIDs to remove from the workspace `fs` folder.
  - **a single cid:** a single IPFS CID to remove from the workspace `fs` folder.
  - **"*"**: special operator. remove ALL files from the workspace `fs` folder.


##### return value

- none

#### examples

##### 1. remove one file

```javascript
await nuron.fs.rm("bafkreibgbn5dua7zep4nyyzbijnlnkhai5jsu25oif2jyz3bi7vwt6u4ce")
```

##### 2. remove multiple files

```javascript
await nuron.fs.rm([
  "bafkreibgbn5dua7zep4nyyzbijnlnkhai5jsu25oif2jyz3bi7vwt6u4ce",
  "bafybeidforebirvjqmtrubrkxccterlhoho2prcgxqlubnmoh5xodccyi4"
])
```

##### 3. remove all files under the fs folder

```javascript
await nuron.fs.rm("*")
```

### pin()

#### syntax

```javascript
let cid = await nuron.fs.pin(cid, callback)
```

##### parameters

- `cid`: can be one of the following:
  - **<cid>:** a single IPFS CID of the file to pin
  - **"*"** pins the entire `fs` folder.
- `callback(update)`: **(optional)** The callback function that gets called every 2 seconds, which gives you a progress update.

The `callback()` method is called every 2 seconds with the following argument:

- `update`
  - `completed`: completed size so far
  - `total`: total size
  - `progress`: the completed/total ratio

##### return value

- `cid`: The IPFS CID of the pinned file, returned after everything has been pinned successfully. (currently supports nft.storage)

#### examples

##### 1. pin all files under the fs folder

```javascript
let response = await nuron.fs.pin("*")
```

##### 2. pin a single file under the fs folder

```javascript
let response = await nuron.fs.pin("bafkreidztp557q7fvbq5t34uat5l3vztkcjzavatrohclmigh5o7qxyrq4")
```

##### 3. pin the entire fs folder with a progress update

```javascript
let response = await nuron.fs.pin("*", (update) => {
  // this will be called every 2 seconds
  console.log("progress", update)
})
```

## db

### write()

Index a token/metadata object in the nuron database.

The database is powered by [Mixtape](https://mixtape.cell.computer) and currently includes two tables:

1. `token`: A C0 token object (either signed or unsigned)
2. `metadata`: An NFT metadata object

> **Note**
>
> the `db.write()` method will **INDEX the objects for efficient filtering**. Basically it's an index, not the original content, so it will look slightly different from the original object.
>
> In order to save and retrieve the EXACT same format, you can use the file system, using the `fs.write()` API to write, and then retrieve later using the `fs.read()` API.
>
> You can store the objects in both the DB and FS, and use the DB for efficient filtering and use the FS for getting the full content of the original data object.

#### syntax

```javascript
await nuron.db.write(table, object)
```

##### parameters

- `table`: The table name to write objects to. Currently supports 2:
  - `token`: A C0 token object (either signed or unsigned)
  - `metadata`: An NFT metadata object
- `object`: a JSON object.
  - In case of inserting into a `token` table, it's an unsigned or signed token
  - In case of inserting into a `metadata` table, it's an NFT metadata object

##### return value

- none

#### examples

##### writing a token object into the token table

```javascript
let signedToken = await nuron.token.create({ cid: cid })
try {
  await nuron.db.write("token", signedToken)
} catch (e) {
  console.log("Error", e)
}
```

##### writing a metadata object into the metadata table

```javascript
let metadata = {
  image: 'ipfs://QmRxrphU6BhpFdC7nnjjTmJLpi2PA6b92XAGY6YuDKvPBq',
  attributes: [
    { trait_type: 'Background', value: 'M1 Aquamarine' },
    { trait_type: 'Fur', value: 'M1 Golden Brown' },
    { trait_type: 'Eyes', value: 'M1 Sleepy' },
    { trait_type: 'Hat', value: 'M1 Cowboy Hat' },
    { trait_type: 'Mouth', value: 'M1 Bored Cigarette' }
  ]
}
try {
  await nuron.db.write("metadata", metadata)
} catch (e) {
  console.log("Error", e)
}
```

### read()

query the database

#### syntax

```javascript
let results = await nuron.db.read(table, query)
```

##### parameters

- `table`: The name of the table to query from. Currently supports `token` and `metadata`
- `query`: The Mixtape JSON query language

##### return value

- `results`: An array of items that match the query

#### examples

##### query tokens

```javascript
// Create some tokens
let signedToken1 = await nuron.token.create({ cid: cid1 })
let signedToken2 = await nuron.token.create({ cid: cid2 })

// Write the tokens to the token table
await nuron.db.write("token", signedToken1)
await nuron.db.write("token", signedToken2)

// Read all items from the token table
let items = await nuron.token.read("token", {
  select: ["*"]
})
```

##### query metadata

```javascript
// Create some nft metadata objects
let meta1 = {
  image: 'ipfs://QmdyyHPpr5VLzFKrPMSWsPAHqVk7sNF9LcAubk3M7JEgvz',
  attributes: [
    { trait_type: 'Background', value: 'M2 Orange' },
    { trait_type: 'Fur', value: 'M2 White' },
    { trait_type: 'Eyes', value: 'M2 Angry' },
    { trait_type: 'Clothes', value: 'M2 Tanktop' },
    { trait_type: 'Hat', value: 'M2 Short Mohawk' },
    { trait_type: 'Mouth', value: 'M2 Dumbfounded' }
  ]  
}
let meta2 = {
  image: 'ipfs://QmbpYzc2Xa5XrtK8V2ioviCuTiHkeh9DFSWibuA5LiNzP5',
  attributes: [
    { trait_type: 'Background', value: 'M1 Blue' },
    { trait_type: 'Fur', value: 'M1 Black' },
    { trait_type: 'Eyes', value: 'M1 Sleepy' },
    { trait_type: 'Clothes', value: 'M1 Work Vest' },
    { trait_type: 'Hat', value: 'M1 Bayc Hat Black' },
    { trait_type: 'Mouth', value: 'M1 Bored Unshaven Cigarette' }
  ]
}
let meta3 = {
  image: 'ipfs://QmaqytYW4XeJPULKawJZeY4A6GtMomqQGmckmYH5Cq2XSQ',
  attributes: [
    { trait_type: 'Background', value: 'M2 Purple' },
    { trait_type: 'Fur', value: 'M2 Brown' },
    { trait_type: 'Eyes', value: 'M2 Eyepatch' },
    { trait_type: 'Clothes', value: 'M2 Lab Coat' },
    { trait_type: 'Mouth', value: 'M2 Bored Unshaven Kazoo' },
    { trait_type: 'Earring', value: 'M2 Silver Hoop' }
  ]
}

// Write the metadata objects to the metadata table
await nuron.db.write("metadata", meta1)
await nuron.db.write("metadata", meta2)
await nuron.db.write("metadata", meta3)

// Read all items that match "Fur: M2 White" from the metadata table
let items = await nuron.token.read("token", {
  select: ["*"],
  where: {
    Fur:  "M2 White"
  }
})
```

### readOne()

query the database and return the first result only.

Same as `read()` but instead of returning an array, returns one object

#### syntax

```javascript
let result = await nuron.db.readOne(table, query)
```

##### parameters

- `table`: The name of the table to query from. Currently supports `token` and `metadata`
- `query`: The Mixtape JSON query language

##### return value

- `result`:
  - if there was a match, returns the matched result
  - if there was NO match, returns `null`

#### examples

##### get one token by metadata cid

```javascript
// Create some tokens
let signedToken = await nuron.token.create({ cid: cid })

// Write the tokens to the token table
await nuron.db.write("token", signedToken)

// Read all items from the token table
let retrievedSignedToken = await nuron.token.readOne("token", {
  select: ["*"],
  where: {
    cid: cid
  }
})

// The signedToken and retrievedSignedToken 
console.log(signedToken, retrievedSignedToken)
```

##### query metadata

```javascript
// Create some nft metadata objects
let meta1 = {
  image: 'ipfs://QmdyyHPpr5VLzFKrPMSWsPAHqVk7sNF9LcAubk3M7JEgvz',
  attributes: [
    { trait_type: 'Background', value: 'M2 Orange' },
    { trait_type: 'Fur', value: 'M2 White' },
    { trait_type: 'Eyes', value: 'M2 Angry' },
    { trait_type: 'Clothes', value: 'M2 Tanktop' },
    { trait_type: 'Hat', value: 'M2 Short Mohawk' },
    { trait_type: 'Mouth', value: 'M2 Dumbfounded' }
  ]  
}
let meta2 = {
  image: 'ipfs://QmbpYzc2Xa5XrtK8V2ioviCuTiHkeh9DFSWibuA5LiNzP5',
  attributes: [
    { trait_type: 'Background', value: 'M1 Blue' },
    { trait_type: 'Fur', value: 'M1 Black' },
    { trait_type: 'Eyes', value: 'M1 Sleepy' },
    { trait_type: 'Clothes', value: 'M1 Work Vest' },
    { trait_type: 'Hat', value: 'M1 Bayc Hat Black' },
    { trait_type: 'Mouth', value: 'M1 Bored Unshaven Cigarette' }
  ]
}
let meta3 = {
  image: 'ipfs://QmaqytYW4XeJPULKawJZeY4A6GtMomqQGmckmYH5Cq2XSQ',
  attributes: [
    { trait_type: 'Background', value: 'M2 Purple' },
    { trait_type: 'Fur', value: 'M2 Brown' },
    { trait_type: 'Eyes', value: 'M2 Eyepatch' },
    { trait_type: 'Clothes', value: 'M2 Lab Coat' },
    { trait_type: 'Mouth', value: 'M2 Bored Unshaven Kazoo' },
    { trait_type: 'Earring', value: 'M2 Silver Hoop' }
  ]
}

// Write the metadata objects to the metadata table
await nuron.db.write("metadata", meta1)
await nuron.db.write("metadata", meta2)
await nuron.db.write("metadata", meta3)

// Read all items that match "Fur: M2 White" from the metadata table
let items = await nuron.token.read("token", {
  select: ["*"],
  where: {
    Fur:  "M2 White"
  }
})
```


### rm()

#### syntax

```javascript
await nuron.db.rm(table, where)
```

##### parameters

- `table`: the table name to remove objects from. Currently supports 2:
  - `token`: the token table
  - `metadata`: the metadata table, stores and indexes arbirary metadata objects
- `where`: a [WHERE](https://mixtape.cell.computer/#/?id=quotwherequot-clause) component of the [Mixtape query language](https://mixtape.cell.computer/#/?id=mixtape-built-in-query-language)

##### return value

- none


#### examples

##### removing all token items

```javascript
// the "{}" clause means no "WHERE" clause, therefore matching all items on the "token" table 
await nuron.db.rm("token", {})
```

##### removing a token with a cid

```javascript
// matches all tokens with the metadata cid of "cid"
await nuron.db.rm("token", {
  cid: cid
})
```

## web


### build()

Build a static site that renders all the tokens and the minting pages

#### syntax

```javascript
await nuron.web.build(config)
```

##### parameters

- `config`: optional configuration parameter
  - `templates`: an array of template file mappings (optional)

##### return value

- none


#### examples

##### default templates

by default you do not have to pass any specific payload. nuron will automatically build the minting site using its own default templates.

```javascript
await nuron.web.build()
```

##### custom templates

You can specify custom templates for `index.html` and `token.html`:


```javascript
await nuron.web.build({
  templates: [{
    src: "https://templateserver.com/index.ejs",      // example EJS template file location
    dest: "index.html"                                // renders to the index.html file
  }, {
    src: "https://templateserver.com/token.ejs",      // example EJS template file location
    dest: "token.html"                                // renders to the token.html file
  }]
})
```

