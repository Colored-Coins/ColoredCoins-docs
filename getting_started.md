# Getting Started with ColoredCoins

Thank you for your interest in ColoredCoins! In this guide, we will explain how to quickly set up and start using the open source ColoredCoins SDK.

The SDK is meant to be used in your own apps, in order to enable creating and transacting digital assets using the [ColoredCoins protocol](https://github.com/Colored-Coins/Colored-Coins-Protocol-Specification). It's a JavaScript SDK, which can be used in a browser or in a Node.js app. You can use the SDK in a client app, allowing all users to manipulate their tokens securely with their own keys, or you can deploy it on a server-side app and manage transactions on your trusted server.

We will be using Node.js in this guide. Please make sure [you have it installed](https://nodejs.org/en/) before continuing!

## Installing the SDK

Let's start by creating a directory for our project. Try something like:
```
mkdir HelloColoredCoins && cd HelloColoredCoins
```

Next we'll create a package.json file. You can do that manually if you know what you're doing, or just:
```
npm init
```
And answer all the questions.

Finally, we'll add the ColoredCoins SDK to our package.json file and install:
```
npm install --save coloredcoins-sdk
```

By default, the SDK is set up to use our Community-Hosted Node of ColoredCoins Core, which validates transactions and allows inspecting the state of the network. So there's no need to install anything else, you can start coding! After you gain some confidence with the SDK, we encourage you to install your own instance of ColoredCoins Core in a controlled environment and use that, for improved security. We'll be releasing ColoredCoins Core very soon, but if you'll want to set up your own backend immediately after completing the guide, you may run our [Docker Node](https://github.com/Colored-Coins/colored-coins-docker-node) which is less optimized and more resource-intensive, but fills the same part. You'll find more information about it [here](https://github.com/Colored-Coins/colored-coins-docker-node).

## Initialization

Let's create a file named `index.js` and open that in your favorite editor.
We'll start by initializing the SDK object, specifying your *mnemonic* and *network*.

### What's a mnemonic?

ColoredCoins is using the Bitcoin network, and in Bitcoin each stakeholder in the system has an identity, which is determined by a key which has two parts, a public and a private part. The private key must be kept secret and safe, you can think of it as a password. From that private key, a matching public key is created and you can think of it as your address, or username, which others will use to send funds to you. You can safely share that address.

The "mnemonic" is a random list of 12 words that will be automatically by the SDK, representing a *private seed*, from which many *private keys* are created, each with its own *public address*. We allow multiple addresses per user for flexibility, but you can think of the *mnemonic* as a private password that represents an identity in the system.

You can read more about these concepts in the followin BIPs (Bitcoin Improvement Proposals): [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) which covers creating the mnemonic itself, and [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) and [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) which define Hierarchical Deterministic Wallets, a method to derive multiple keys from a single seed.

### What's a network?

Bitcoin transactions have a cost attached - miner fees. These can range from a few cents to a Dollar or more, based on current traffic in the network. To allow development and testing without paying those fees, there's a "test network" for Bitcoin, called `testnet` for short. This network isn't secure for real-value transactions, but you can transact for free. Transactions still have fees, but the fees are paid in "test bitcoins", which you can get for free from "Testnet Bitcoin Faucets", websites that will send you free test bitcoins to help with development. You can search for some on Google, or try [this one](http://tpfaucet.appspot.com/).

One your app is ready for production, you'll switch it to `mainnet`, the "real money" Bitcoin network.

### Generating your mnemonic for the first time

Go ahead and type the following into your `index.js` file:

```javascript
var ColoredCoins = require('coloredcoins-sdk')

var settings = {
  network: 'testnet',
  mnemonic: null
}

var cc = new ColoredCoins(settings)

cc.on('connect', function() {
  console.log("mnemonic: ", cc.hdwallet.getMnemonic())
})

cc.init()

```

We create a `settings` object, specifying the `testnet` network, and no mnemonic. This means a new mnemonic will be generated for us.
We than initailize a `cc` SDK object with the settings, and register the `connect` event with a function that will print out the mnemonic.
Finially we run `cc.init()` to tell the SDK to connect to our servers. Once connected, it will fire the `connect` event and print the mnemonic.

You can run this code with `node index.js`.

In a real app, you would now store that mnemonic in a safe place, and use that same mnemonic the next time you initialize the ColoredCoins SDK. For the purpose of this guide, just copy the mnemonic into the settings object, like so (but do it with your own mnemonic!):

```javascript
var settings = {
  network: 'testnet',
  mnemonic: 'unusual nominee adjust tank pave index picnic various exile fluid deny razor'
}
```

Now, every time you run this code, you'll use the same identity.

## Issuing a digital asset

Let's say you're in charge of the Central Bank of The Republic of La La Land. You want LaLaLandians to use your own currency, the La La Lira. Since La La Land is a modern country, you're not going to fool around with paper-based cash, and decide to go all-digital with the ColoredCoins protocol.

To start off, you'll create 500 units of the new asset.

Go ahead and change your code:

```javascript
cc.on('connect', function() {

  var asset = {
      amount: 500,
      metadata: {
          assetName: 'La La Lira',
          issuer: 'La La Land Central Bank',
          description: "The national currency of the People's Republic of La La Land"
      },
      fee: 1000,
      issueAddress: cc.hdwallet.getAddress(0, 0)
  }

  cc.issueAsset(asset, function (err, body) {
    if (err) return console.error(err)
      console.log("Body: ",body)
  })

})
```

With the `asset` object, we're specifying a new asset. It will have 500 units (so 500 La La Liras). It also specifies some metadata. The metadata could be anything you'd like. We chose to use some standardized fields that give context to what that asset is, but you don't have to use metadata if you don't want to.
The specified `fee` is how much you're going to pay for the miner fee on that transaction. It's denominated in "satoshis", and there are 100,000,000 "satoshis" in one bitcoin. Remember, this is on `testnet` right now, so this is all "play money".
We also specify the address that will issue the asset. This address will be considered "the issuer", and people are expected to trust the identity that is issuing an asset. With `(0, 0)` we just pick the first address that can't be generated from the mnemonic. This address must have enough bitcoin to pay for the specified miner's fee.

We then use `issueAsset()` to send the request to the ColoredCoins node and issue the asset. When the request returns, we print the body.

Run it!

You'll see an error like this:
```
{ explanation: 'address mzjtnyfYxZYL9nVzmdn5eBJyD18M8joUhS does not have any unused outputs',
  code: 20003,
  status: 500,
  name: 'NotEnoughFundsError',
  message: 'Not enough satoshi to cover transaction',
  type: 'issuance' }
```

We get this error because we didn't send enough bitcoins to cover the fee to this address. Go ahead and Google for a Bitcoin Testnet faucet (you can try [this one](http://tpfaucet.appspot.com/)) and ask for coins to be sent to your address (the one in the error you received, not the one in the example above). Now run the code again. This time you'll get a response like this:

```
Body:  { txHex: '010000000165e150aefa77c085e54a5000a76f0c2c09d645a57dd4cadb799a1d9072d28365020000006a473044022079555c0df188771a0ca153b8890b982a9aecdc8132c21af4bdeccf29367d6fbd022023620212de627ae0ebaabe4c07371e4f1f9184ef50763a511b4ae18d4f4bc12b0121039efc0bb7d8d4374cc696023e71fc39e6bee6565f4065aed0a3717e81c38bce9affffffff0300000000000000003d6a3b43430201a2c1f99866f7e843dad04b47dd85dc761027c3a14edbef1c024e9dd38729dd5544b642af1985158aea7d85acd6b4b80d868b051320521090480301000000001976a914d2dcdfe8bd61c2313852b5f19f70a22988b50e5e88ac58020000000000001976a914d2dcdfe8bd61c2313852b5f19f70a22988b50e5e88ac00000000',
  assetId: 'La6r7PVFttyVYMERnwHSpUfpQJrbkUHQtSpRrT',
  coloredOutputIndexes: [ 2 ],
  txid: 'e97f5a04fe328a0d0df2d340ad7aed5b89f616e28878309dd00735a52d46aadd',
  receivingAddresses: [],
  issueAddress: 'mzjtnyfYxZYL9nVzmdn5eBJyD18M8joUhS' }
```

The important part is the `assetId`. This is the ID of your newly-issued asset and this is what you will use to identify it in your app. You can launch the [ColoredCoins Block Explorer](http://coloredcoins.org/explorer/testnet) and enter the Asset ID you got in the response, and look at its details, like how many were created, its name, and so on. Congrats!

### Making an asset divisible

One problem with our asset, is that we can only send full units of it. We can't send 0.5 a Lira to someone. That's a problem! We want to have 100 La La cents in a La La Lira, so we need to be able to divide it.
We can specify an asset with a divisibility of **2**, which means we can have up to 2 digits after the decimal point (like 0.99). We can specify divisibility from **0** up to **8**, with 0 being the default, meaning units can't be divided at all.

Let's change the definition of the asset:

```javascript
var asset = {
    amount: 50000,
    metadata: {
        assetName: 'La La Lira',
        issuer: 'La La Land Central Bank',
        description: "The national currency of the People's Republic of La La Land"
    },
    divisibility: 2,
    fee: 1000,
    issueAddress: cc.hdwallet.getAddress(0, 0)
}
```

One important thing to note, is that the specified `amount` is denominated in the smallest fraction, which is now the La La cent. So we changed the amount from 500 to 50,000, as we still want to issue 500 full La La Liras.

When we run the code, we'll create a new asset class, with a new Asset ID. We can create as many Asset IDs per issuing address as we want (but more on that later).

### Issuing more units of an asset

OK, so we did a great job as a Central Bank, and the La La Land economy is growing. We need to issue 50 more Liras to satisfy our inflation goals. But how can we do that? We saw in the previous section that when we issue again, a new Asset ID is created. How can we issue more units of the same asset?

This brings us to **Locked** and **Unlocked** assets. Assets are **Locked** by default, which means once they were issued, you can *never* issue more units of the same asset. For digital currency, this might not be ideal, so we offer issuance of **Unlocked** assets. With Unlocked assets, you can always issue more units, but only if you control the original issuing address.

Keep in mind, while an address can issue as many different Locked asset types as needed, you can only create one **Unlocked** asset type per address. Every subsequent **Unlocked** issuance with the same address will issue more units of that same asset. If you need to issue multiple unlocked assets, you can use other addresses that can be generated from your mnemonic by the SDK.

Here's the updated asset definition:
```javascript
var asset = {
    amount: 50000,
    metadata: {
        assetName: 'La La Lira',
        issuer: 'La La Land Central Bank',
        description: "The national currency of the People's Republic of La La Land"
    },
    divisibility: 2,
    fee: 1000,
    reissueable: true,
    issueAddress: cc.hdwallet.getAddress(0, 0)
}
```

Note we now have `reissuable: true` to signify that this is an unlocked asset. You can run this code as many times as you want. The first time, you'll get a new Asset ID, and the next times you'll get the same Asset ID as the first time. Look this Asset ID up in the [ColoredCoins Block Explorer](http://coloredcoins.org/explorer/testnet), and you'll see that more units are being issued each time you run it.

### Sending during issuance

In the next section we'll cover how citizens of La La Land can send Liras between each other. But first, if you as the issuer want to send some of the newly issued units to another address, you can do that within the issuance transaction itself, like so:

```javascript
var asset = {
    amount: 50000,
    metadata: {
        assetName: 'La La Lira',
        issuer: 'La La Land Central Bank',
        description: "The national currency of the People's Republic of La La Land"
    },
    divisibility: 2,
    fee: 1000,
    reissueable: true,
    transfer: [{
      address: 'mvaHph557j63CyJxmEaJ8F38SC3cNetvaa', amount: 300
    }],
    issueAddress: cc.hdwallet.getAddress(0, 0)
}
```

Note the `transfer` array, specifying an address and amount to send to that address. You can add more if you want. When you run this code, 3.00 new Liras will be sent to that address. Look it up in the [Block Explorer](http://coloredcoins.org/explorer/testnet)! You'll see a transaction on the Asset page, or you can look for the address itself and see that it received some Liras.

## Sending asset units

Sending some La La Liras is easy with the ColoredCoins SDK.
First, make sure you issued an asset in the last section, and take note of the returned Asset ID. Then, you can use the following code:

```javascript
cc.on('connect', function() {

  var assetId = 'YOUR_ASSET_ID_HERE'
  var address1 = 'mgrhPJzH37YctFsNtZGAHgvgEhJQ2A62ss'
  var address2 = 'mfhdwaZd9csRDGVPBGVZeup45JpEGvYqhA'

  var send = {
    from: [cc.hdwallet.getAddress(0, 0)],
    to: [{
      address: 'mgrhPJzH37YctFsNtZGAHgvgEhJQ2A62ss',
      assetId: assetId,
      amount: 1
    }, {
      address: 'mfhdwaZd9csRDGVPBGVZeup45JpEGvYqhA',
      assetId: assetId,
      amount: 2
    }],
    fee: 1000
  }

  cc.sendAsset(send, function (err, body) {
    if (err) return console.error(err)
      console.log("Body: ",body)
  })
})
```

Make sure to enter your own Asset ID in the appropriate variable. Also note that you have to specify the address you're sending from, which must have enough Liras to send. You can even send to mutiple recipients in the same transactions if you want.

You'll get a response like this:

```
Body:  { txHex: '0100000002ddaa462da53507d09d307888e216f6895bed7aad40d3f20d0d8a32fe045a7fe9020000006b48304502210095a6d3294c859b385efe5499efe93befd4e92d474bcbd208ba1a6bb997cff2610220792826056280617e34f43700c5c5b346b606673b2e77abe7fa1350ffe53c93140121039efc0bb7d8d4374cc696023e71fc39e6bee6565f4065aed0a3717e81c38bce9affffffffddaa462da53507d09d307888e216f6895bed7aad40d3f20d0d8a32fe045a7fe9010000006a473044022079f5d70d78f5520b33e991a723b12842c11490b78615b085ce5e892cb71a90d902203963c9a3fed38b5d00f9dca52657b15c839210b68857056adb427084d7efea7d0121039efc0bb7d8d4374cc696023e71fc39e6bee6565f4065aed0a3717e81c38bce9affffffff0558020000000000001976a9140eb3ffd2c449f56cfdf7f9cb8179fbcabce9c61a88ac58020000000000001976a91402054158a7b64cc8a06827278145b880c3453b8b88ac00000000000000000a6a084343021500010102f83f0301000000001976a914d2dcdfe8bd61c2313852b5f19f70a22988b50e5e88ac58020000000000001976a914d2dcdfe8bd61c2313852b5f19f70a22988b50e5e88ac00000000',
  multisigOutputs: [],
  coloredOutputIndexes: [ 0, 1, 4 ],
  txid: '940438356b489ae8d6dda1ee1475db1089c41accfeb1e334e7d7a8d598f67ec0' }
```

Look at the `txid` property from your response - you can copy that into the [Block Explorer](http://coloredcoins.org/explorer/testnet) and see your transaction.

## Burning asset units

With ColoredCoins, anyone holding units of an asset can `burn` (destroy) them. In the case of the La La Lira, it would be irrational for most users to burn their asset (maybe one citizen feels money is the root of all evil and wants to get rid of as much as he can...). But as the Central Bank, perhaps you'd like to burn some Liras you control, to reduce the money supply. Burning is easy, and you can do that with either Locked or Unlocked assets:

```javascript
cc.on('connect', function() {

  var args = {
    from: [cc.hdwallet.getAddress(0, 0)],
    burn: [{
      assetId: 'YOUR_ASSET_HERE',
      amount: 5
    }],
    fee: 1000
  }

  cc.burnAsset(args, function (err, body) {
    if (err) return console.error(err)
      console.log("Body: ", body)
  })
})
```

As before, you must specify a miner's fee, your asset's ID, and a `from` address (or array of addresses) to `burn` units from. So this address must hold units of course. You'll get a response like this:

```
Body:  { txHex: '0100000002c07ef698d5a8d7e734e3b1fecc1ac48910db7514eea1ddd6e89a486b35380494040000006b483045022100a9e64b0210cfa71bc0a31db5f301b71fc77b112096d94a3b7d4019c63168fc3702203d1b9e4c11e245dd981a8ab845c80aee7dd6d3f0ec03f68307ccb2211a2b9e060121039efc0bb7d8d4374cc696023e71fc39e6bee6565f4065aed0a3717e81c38bce9affffffffc07ef698d5a8d7e734e3b1fecc1ac48910db7514eea1ddd6e89a486b35380494030000006b4830450221008b4e200bf8efb0e522587ab47903ba05262b1c169e504ebedd3d86675ee9a71c02203b60b4e0b49eaec98f7483e39a11da48f58e46fd6fde7c7424128adb8ab315820121039efc0bb7d8d4374cc696023e71fc39e6bee6565f4065aed0a3717e81c38bce9affffffff030000000000000000086a06434302251f05103c0301000000001976a914d2dcdfe8bd61c2313852b5f19f70a22988b50e5e88ac58020000000000001976a914d2dcdfe8bd61c2313852b5f19f70a22988b50e5e88ac00000000',
  multisigOutputs: [],
  coloredOutputIndexes: [ 2 ],
  txid: 'c1c0aace5b49aa8660078bc349dc71bd57e8af5994e69d99725786a54a0d683a' }
```

With the [Block Explorer](http://coloredcoins.org/explorer/testnet), you can search for your address or Asset ID, and see that units were burned. You can also look at the `txid` from the response and see the transaction that burns units.

## Querying for asset holders

One of the advantages of the ColoredCoins protocol is that it allows transparency about asset issuance and transactions. This can be useful for both issuers and users. As a central bank, maybe you wish to see who's holding the currency.

Getting a list of holders of La La Lira is easy:

```javascript
var assetId = 'La6r7PVFttyVYMERnwHSpUfpQJrbkUHQtSpRrT'

cc.on('connect', function () {
  cc.getStakeHolders(assetId, function (err, body) {
    if (err) return console.error(err)
    console.log("Body: ",body)
  })
})
```

You can use your own Asset ID. You'll get a response like this:
```
Body:  { assetId: 'La6r7PVFttyVYMERnwHSpUfpQJrbkUHQtSpRrT',
  holders:
   [ { address: 'mgrhPJzH37YctFsNtZGAHgvgEhJQ2A62ss', amount: 1 },
     { address: 'mfhdwaZd9csRDGVPBGVZeup45JpEGvYqhA', amount: 2 },
     { address: 'mzjtnyfYxZYL9nVzmdn5eBJyD18M8joUhS', amount: 492 } ],
  divisibility: 0,
  lockStatus: true,
  aggregationPolicy: 'aggregatable',
  someUtxo: '940438356b489ae8d6dda1ee1475db1089c41accfeb1e334e7d7a8d598f67ec0:0' }
```

You get a list of addresses holding your asset and number of units being held, and also some general details about the asset, like its divisibility, or it being locked or unlocked.

## What's next?

Congratulations! You were able to issue digital currencies for the citizens of La La Land, and they can now transact freely with each other.

You can use what you learned in your own app. Give it a try! Remember that when you're ready to push your app to a production environment, you should be using `mainnet`, and you should be taking care of funding transactions with real bitcoins.

And don't forget to tell us what you built! We're waiting for you at the [ColoredCoins Slack Team](http://slack.coloredcoins.org/).
