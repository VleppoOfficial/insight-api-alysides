# Vleppo Insight API

This repository contains the source code for the Alysides blockchain REST and web socket API service for [Bitcore Node](https://github.com/VleppoOfficial/bitcore-node-alysides).

This is a backend-only service. If you're looking for the web frontend application, take a look at https://github.com/VleppoOfficial/insight-ui-alysides.

## Installation

This section provides full installation instructions of Insight API and all its dependencies.
Note that the Insight UI is not covered by these instructions. Refer to https://github.com/VleppoOfficial/insight-ui-alysides/tree/generic-ui#installation for installing the Insight UI.

### Prerequisites

- [Bitcore Node](https://github.com/VleppoOfficial/bitcore-node-alysides)
- A node running [Vleppo Alysides](https://github.com/VleppoOfficial/alysides)
- GNU/Linux x86_32/x86_64, or OSX 64bit
- Node.js v4 (use [nvm](https://github.com/nvm-sh/nvm) to manage Node.js versions)
- ZeroMQ (libzmq3-dev for Ubuntu/Debian or zeromq on OSX)
- ~200GB of disk storage
- ~8GB of RAM

### Getting Started

1. Make sure you have the correct Node.js version installed: `nvm install v4`
    - **Note:** this command may not work immediately after installing nvm, you may need to reboot shell for changes to take effect.
2. Make sure you have ZeroMQ installed:
    - Linux: `sudo apt install libzmq3-dev`
    - OSX: `brew install zeromq`
3. Install bitcore-node-alysides:\
  `npm install -g https://github.com/VleppoOfficial/bitcore-node-alysides`\
  `bitcore-node create mynode`\
  `cd mynode`
4. Install Insight:\
  `bitcore-node install https://github.com/VleppoOfficial/insight-api-alysides`
5. Make sure to either setup an Alysides instance for your chain locally or have access to a running local or remote Alysides node.
6. In the machine where Alysides is running (can be the same as where bitcore-node is set up), create or edit the ".conf" file, located by default in ".komodo/your_chain_name/". Make sure the file includes the following lines:\
  `rpcuser=<your secure rpc user>`\
  `rpcpassword=<your secure rpc password>`\
  `rpcport=<your chain's rpc port>`\
  `server=1`\
  `txindex=1`\
  `addressindex=1`\
  `timestampindex=1`\
  `spentindex=1`\
  `zmqpubrawtx=tcp://127.0.0.1:28332`\
  `zmqpubhashblock=tcp://127.0.0.1:28332`
    - Make sure port 28332 is not blocked for both the bitcore-node and Alysides machines (including if both are being run on the same machine). The port number can be changed if necessary.
7. In the mynode folder, edit the bitcore-node.json file. Make sure to specify the Alysides node's IP address in rpchost (set to 127.0.0.1 if localhost), and the correct rpcuser, rpcpassword, rpcport and zmqpubrawtx according to the same values as in the node's .conf file.
8. Launch your Alysides node (if connected solely to a local node). 
9. Launch Insight: `bitcore-node start`
    - **Note:** After this, you may want to restart Alysides and use the `-reindex` launch parameter when starting Alysides again in order for Insight to successfully capture past transactions.

The API endpoints will be available by default at: `http://localhost:3001/insight-api-alysides/`.

## API Usage and Documentation

## Notes on Upgrading from v0.3

The unspent outputs format now has `satoshis` and `height`:
```
[
  {
    "address":"mo9ncXisMeAoXwqcV5EWuyncbmCcQN4rVs",
    "txid":"d5f8a96faccf79d4c087fa217627bb1120e83f8ea1a7d84b1de4277ead9bbac1",
    "vout":0,
    "scriptPubKey":"76a91453c0307d6851aa0ce7825ba883c6bd9ad242b48688ac",
    "amount":0.000006,
    "satoshis":600,
    "confirmations":0,
    "ts":1461349425
  },
  {
    "address": "mo9ncXisMeAoXwqcV5EWuyncbmCcQN4rVs",
    "txid": "bc9df3b92120feaee4edc80963d8ed59d6a78ea0defef3ec3cb374f2015bfc6e",
    "vout": 1,
    "scriptPubKey": "76a91453c0307d6851aa0ce7825ba883c6bd9ad242b48688ac",
    "amount": 0.12345678,
    "satoshis: 12345678,
    "confirmations": 1,
    "height": 300001
  }
]
```
The `timestamp` property will only be set for unconfirmed transactions and `height` can be used for determining block order. The `confirmationsFromCache` is nolonger set or necessary, confirmation count is only cached for the time between blocks.

There is a new `GET` endpoint or raw blocks at `/rawblock/<blockHash>`:

Response format:
```
{
  "rawblock": "blockhexstring..."
}
```

There are a few changes to the `GET` endpoint for `/addr/[:address]`:

- The list of txids in an address summary does not include orphaned transactions
- The txids will be sorted in block order
- The list of txids will be limited at 1000 txids
- There are two new query options "from" and "to" for pagination of the txids (e.g. `/addr/[:address]?from=1000&to=2000`)

Some additional general notes:
- The transaction history for an address will be sorted in block order
- The response for the `/sync` endpoint does not include `startTs` and `endTs` as the sync is no longer relevant as indexes are built in komodod.
- The endpoint for `/peer` is no longer relevant connection to komodod is via ZMQ.
- `/tx` endpoint results will now include block height, and spentTx related fields will be set to `null` if unspent.
- `/block` endpoint results does not include `confirmations` and will include `poolInfo`.

## Notes on Upgrading from v0.2

Some of the fields and methods are not supported:

The `/tx/<txid>` endpoint JSON response will not include the following fields on the "vin"
object:
- `doubleSpentTxId` // double spends are not currently tracked
- `isConfirmed` // confirmation of the previous output
- `confirmations` // confirmations of the previous output
- `unconfirmedInput`

The `/tx/<txid>` endpoint JSON response will not include the following fields on the "vout"
object.
- `spentTs`

The `/status?q=getTxOutSetInfo` method has also been removed due to the query being very slow and locking komodod.

Plug-in support for Insight API is also no longer available, as well as the endpoints:
- `/email/retrieve`
- `/rates/:code`

Caching support has not yet been added in the v0.3 upgrade.

## Query Rate Limit

To protect the server, insight-api-alysides has a built it query rate limiter. It can be configurable in `bitcore-node.json` with:
``` json
  "servicesConfig": {
    "insight-api-alysides": {
      "rateLimiterOptions": {
        "whitelist": ["::ffff:127.0.0.1"]
      }
    }
  }
```
With all the configuration options available: https://github.com/bitpay/insight-api/blob/master/lib/ratelimiter.js#L10-17

Or disabled entirely with:
``` json
  "servicesConfig": {
    "insight-api-alysides": {
      "disableRateLimiter": true
    }
  }
  ```
  

## API HTTP Endpoints

### Block
```
  /insight-api-alysides/block/[:hash]
  /insight-api-alysides/block/00000000a967199a2fad0877433c93df785a8d8ce062e5f9b451cd1397bdbf62
```

### Block Index
Get block hash by height
```
  /insight-api-alysides/block-index/[:height]
  /insight-api-alysides/block-index/0
```
This would return:
```
{
  "blockHash":"000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f"
}
```
which is the hash of the Genesis block (0 height)


### Raw Block
```
  /insight-api-alysides/rawblock/[:blockHash]
  /insight-api-alysides/rawblock/[:blockHeight]
```

This would return:
```
{
  "rawblock":"blockhexstring..."
}
```

### Block Summaries

Get block summaries by date:
```
  /insight-api-alysides/blocks?limit=3&blockDate=2016-04-22
```

Example response:
```
{
  "blocks": [
    {
      "height": 408495,
      "size": 989237,
      "hash": "00000000000000000108a1f4d4db839702d72f16561b1154600a26c453ecb378",
      "time": 1461360083,
      "txlength": 1695,
      "poolInfo": {
        "poolName": "BTCC Pool",
        "url": "https://pool.btcc.com/"
      }
    }
  ],
  "length": 1,
  "pagination": {
    "next": "2016-04-23",
    "prev": "2016-04-21",
    "currentTs": 1461369599,
    "current": "2016-04-22",
    "isToday": true,
    "more": true,
    "moreTs": 1461369600
  }
}
```

### Transaction
```
  /insight-api-alysides/tx/[:txid]
  /insight-api-alysides/tx/525de308971eabd941b139f46c7198b5af9479325c2395db7f2fb5ae8562556c
  /insight-api-alysides/rawtx/[:rawid]
  /insight-api-alysides/rawtx/525de308971eabd941b139f46c7198b5af9479325c2395db7f2fb5ae8562556c
```

### Address
```
  /insight-api-alysides/addr/[:addr][?noTxList=1][&from=&to=]
  /insight-api-alysides/addr/mmvP3mTe53qxHdPqXEvdu8WdC7GfQ2vmx5?noTxList=1
  /insight-api-alysides/addr/mmvP3mTe53qxHdPqXEvdu8WdC7GfQ2vmx5?from=1000&to=2000
```

### Address Properties
```
  /insight-api-alysides/addr/[:addr]/balance
  /insight-api-alysides/addr/[:addr]/totalReceived
  /insight-api-alysides/addr/[:addr]/totalSent
  /insight-api-alysides/addr/[:addr]/unconfirmedBalance
```
The response contains the value in Satoshis.

### Unspent Outputs
```
  /insight-api-alysides/addr/[:addr]/utxo
```
Sample return:
```
[
  {
    "address":"mo9ncXisMeAoXwqcV5EWuyncbmCcQN4rVs",
    "txid":"d5f8a96faccf79d4c087fa217627bb1120e83f8ea1a7d84b1de4277ead9bbac1",
    "vout":0,
    "scriptPubKey":"76a91453c0307d6851aa0ce7825ba883c6bd9ad242b48688ac",
    "amount":0.000006,
    "satoshis":600,
    "confirmations":0,
    "ts":1461349425
  },
  {
    "address": "mo9ncXisMeAoXwqcV5EWuyncbmCcQN4rVs",
    "txid": "bc9df3b92120feaee4edc80963d8ed59d6a78ea0defef3ec3cb374f2015bfc6e",
    "vout": 1,
    "scriptPubKey": "76a91453c0307d6851aa0ce7825ba883c6bd9ad242b48688ac",
    "amount": 0.12345678,
    "satoshis: 12345678,
    "confirmations": 1,
    "height": 300001
  }
]
```

### Unspent Outputs for Multiple Addresses
GET method:
```
  /insight-api-alysides/addrs/[:addrs]/utxo
  /insight-api-alysides/addrs/2NF2baYuJAkCKo5onjUKEPdARQkZ6SYyKd5,2NAre8sX2povnjy4aeiHKeEh97Qhn97tB1f/utxo
```

POST method:
```
  /insight-api-alysides/addrs/utxo
```

POST params:
```
addrs: 2NF2baYuJAkCKo5onjUKEPdARQkZ6SYyKd5,2NAre8sX2povnjy4aeiHKeEh97Qhn97tB1f
```

### Transactions by Block
```
  /insight-api-alysides/txs/?block=HASH
  /insight-api-alysides/txs/?block=00000000fa6cf7367e50ad14eb0ca4737131f256fc4c5841fd3c3f140140e6b6
```
### Transactions by Address
```
  /insight-api-alysides/txs/?address=ADDR
  /insight-api-alysides/txs/?address=mmhmMNfBiZZ37g1tgg2t8DDbNoEdqKVxAL
```

### Transactions for Multiple Addresses
GET method:
```
  /insight-api-alysides/addrs/[:addrs]/txs[?from=&to=]
  /insight-api-alysides/addrs/2NF2baYuJAkCKo5onjUKEPdARQkZ6SYyKd5,2NAre8sX2povnjy4aeiHKeEh97Qhn97tB1f/txs?from=0&to=20
```

POST method:
```
  /insight-api-alysides/addrs/txs
```

POST params:
```
addrs: 2NF2baYuJAkCKo5onjUKEPdARQkZ6SYyKd5,2NAre8sX2povnjy4aeiHKeEh97Qhn97tB1f
from (optional): 0
to (optional): 20
noAsm (optional): 1 (will omit script asm from results)
noScriptSig (optional): 1 (will omit the scriptSig from all inputs)
noSpent (option): 1 (will omit spent information per output)
```

Sample output:
```
{ totalItems: 100,
  from: 0,
  to: 20,
  items:
    [ { txid: '3e81723d069b12983b2ef694c9782d32fca26cc978de744acbc32c3d3496e915',
       version: 1,
       locktime: 0,
       vin: [Object],
       vout: [Object],
       blockhash: '00000000011a135e5277f5493c52c66829792392632b8b65429cf07ad3c47a6c',
       confirmations: 109367,
       time: 1393659685,
       blocktime: 1393659685,
       valueOut: 0.3453,
       size: 225,
       firstSeenTs: undefined,
       valueIn: 0.3454,
       fees: 0.0001 },
      { ... },
      { ... },
      ...
      { ... }
    ]
 }
```

Note: if pagination params are not specified, the result is an array of transactions.

### Transaction Broadcasting
POST method:
```
  /insight-api-alysides/tx/send
```
POST params:
```
  rawtx: "signed transaction as hex string"

  eg

  rawtx: 01000000017b1eabe0209b1fe794124575ef807057c77ada2138ae4fa8d6c4de0398a14f3f00000000494830450221008949f0cb400094ad2b5eb399d59d01c14d73d8fe6e96df1a7150deb388ab8935022079656090d7f6bac4c9a94e0aad311a4268e082a725f8aeae0573fb12ff866a5f01ffffffff01f0ca052a010000001976a914cbc20a7664f2f69e5355aa427045bc15e7c6c77288ac00000000

```
POST response:
```
  {
      txid: [:txid]
  }

  eg

  {
      txid: "c7736a0a0046d5a8cc61c8c3c2821d4d7517f5de2bc66a966011aaa79965ffba"
  }
```

### Historic Blockchain Data Sync Status
```
  /insight-api-alysides/sync
```

### Live Network P2P Data Sync Status
```
  /insight-api-alysides/peer
```

### Status of the Alysides Network
```
  /insight-api-alysides/status?q=xxx
```

Where "xxx" can be:

 * getInfo
 * getDifficulty
 * getBestBlockHash
 * getLastBlockHash


### Utility Methods
```
  /insight-api-alysides/utils/estimatefee[?nbBlocks=2]
```


## Web Socket API
The web socket API is served using [socket.io](http://socket.io).

The following are the events published by insight:

`tx`: new transaction received from network. This event is published in the 'inv' room. Data will be a app/models/Transaction object.
Sample output:
```
{
  "txid":"00c1b1acb310b87085c7deaaeba478cef5dc9519fab87a4d943ecbb39bd5b053",
  "processed":false
  ...
}
```


`block`: new block received from network. This event is published in the `inv` room. Data will be a app/models/Block object.
Sample output:
```
{
  "hash":"000000004a3d187c430cd6a5e988aca3b19e1f1d1727a50dead6c8ac26899b96",
  "time":1389789343,
  ...
}
```

`<komodoAddress>`: new transaction concerning <komodoAddress> received from network. This event is published in the `<komodoAddress>` room.

`status`: every 1% increment on the sync task, this event will be triggered. This event is published in the `sync` room.

Sample output:
```
{
  blocksToSync: 164141,
  syncedBlocks: 475,
  upToExisting: true,
  scanningBackward: true,
  isEndGenesis: true,
  end: "000000000933ea01ad0ee984209779baaec3ced90fa3f408719526f8d77f4943",
  isStartGenesis: false,
  start: "000000009f929800556a8f3cfdbe57c187f2f679e351b12f7011bfc276c41b6d"
}
```

### Example Usage

The following html page connects to the socket.io insight API and listens for new transactions.

html
```
<html>
<body>
  <script src="http://<insight-server>:<port>/socket.io/socket.io.js"></script>
  <script>
    eventToListenTo = 'tx'
    room = 'inv'

    var socket = io("http://<insight-server>:<port>/");
    socket.on('connect', function() {
      // Join the room.
      socket.emit('subscribe', room);
    })
    socket.on(eventToListenTo, function(data) {
      console.log("New transaction received: " + data.txid)
    })
  </script>
</body>
</html>
```

## License
(The MIT License)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
