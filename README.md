# MOAC Web3 ProviderEngine

MOAC Web3 ProviderEngine is a tool to use for MOAC MASK. It is forked from Web3 ProviderEngine and modified to use on MOAC network. We handle necessary types of RPC to make the MoacMask extension work. 

### Composable

MOAC is compatiable with some ETHEREUM RPC commands and web3 as well.

Built to be modular - works via a stack of 'sub-providers' which are like normal web3 providers but only handle a subset of rpc methods.

The subproviders can emit new rpc requests in order to handle their own;  e.g. `eth_call` may trigger `eth_getAccountBalance`, `eth_getCode`, and others.
The provider engine also handles caching of rpc request results.

```js
const ProviderEngine = require('web3-provider-engine')
const CacheSubprovider = require('web3-provider-engine/subproviders/cache.js')
const FixtureSubprovider = require('web3-provider-engine/subproviders/fixture.js')
const FilterSubprovider = require('web3-provider-engine/subproviders/filters.js')
const VmSubprovider = require('web3-provider-engine/subproviders/vm.js')
const HookedWalletSubprovider = require('web3-provider-engine/subproviders/hooked-wallet.js')
const NonceSubprovider = require('web3-provider-engine/subproviders/nonce-tracker.js')
const RpcSubprovider = require('web3-provider-engine/subproviders/rpc.js')

var engine = new ProviderEngine()
var web3 = new Web3(engine)

// static results
engine.addProvider(new FixtureSubprovider({
  web3_clientVersion: 'ProviderEngine/v0.0.0/javascript',
  net_listening: true,
  eth_hashrate: '0x00',
  eth_mining: false,
  eth_syncing: true,
}))

// cache layer
engine.addProvider(new CacheSubprovider())

// filters
engine.addProvider(new FilterSubprovider())

// pending nonce
engine.addProvider(new NonceSubprovider())

// vm
engine.addProvider(new VmSubprovider())

// id mgmt
engine.addProvider(new HookedWalletSubprovider({
  getAccounts: function(cb){ ... },
  approveTransaction: function(cb){ ... },
  signTransaction: function(cb){ ... },
}))

// data source
engine.addProvider(new RpcSubprovider({
  rpcUrl: 'https://testrpc.metamask.io/',
}))

// log new blocks
engine.on('block', function(block){
  console.log('================================')
  console.log('BLOCK CHANGED:', '#'+block.number.toString('hex'), '0x'+block.hash.toString('hex'))
  console.log('================================')
})

// network connectivity error
engine.on('error', function(err){
  // report connectivity errors
  console.error(err.stack)
})

// start polling for blocks
engine.start()
```

When importing in webpack:
```js
import * as Web3ProviderEngine  from 'web3-provider-engine';
import * as RpcSource  from 'web3-provider-engine/subproviders/rpc';
import * as HookedWalletSubprovider from 'web3-provider-engine/subproviders/hooked-wallet';
```

### Built For Zero-Clients

The [Ethereum JSON RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC) was not designed to have one node service many clients.
However a smaller, lighter subset of the JSON RPC can be used to provide the blockchain data that an Ethereum 'zero-client' node would need to function.
We handle as many types of requests locally as possible, and just let data lookups fallback to some data source ( hosted rpc, blockchain api, etc ).
Categorically, we don’t want / can’t have the following types of RPC calls go to the network:
* id mgmt + tx signing (requires private data)
* filters (requires a stateful data api)
* vm (expensive, hard to scale)

### Change Log

##### 0.1.0

  Modified from MetaMask provider-engine 14.0.3
- Changed the ethereumjs-tx library to moac-tx library
- Some of the web3 rpc methods were removed: shh
- Some tests were not passed 
- We will change the library to Chain3.


### Current RPC method support:

##### static
- [x] web3_clientVersion
- [x] net_version
- [x] net_listening
- [x] net_peerCount
- [x] eth_protocolVersion
- [x] eth_hashrate
- [x] eth_mining
- [x] eth_syncing

##### filters
- [x] eth_newBlockFilter
- [x] eth_newPendingTransactionFilter
- [x] eth_newFilter
- [x] eth_uninstallFilter
- [x] eth_getFilterLogs
- [x] eth_getFilterChanges

##### accounts manager
- [x] eth_coinbase
- [x] eth_accounts
- [x] eth_sendTransaction
- [x] eth_sign
- [x] [eth_signTypedData](https://github.com/ethereum/EIPs/pull/712)

##### vm
- [x] eth_call
- [x] eth_estimateGas

##### db source
- [ ] db_putString
- [ ] db_getString
- [ ] db_putHex
- [ ] db_getHex


##### data source ( fallback to rpc )
* eth_gasPrice
* eth_blockNumber
* eth_getBalance
* eth_getBlockByHash
* eth_getBlockByNumber
* eth_getBlockTransactionCountByHash
* eth_getBlockTransactionCountByNumber
* eth_getCode
* eth_getStorageAt
* eth_getTransactionByBlockHashAndIndex
* eth_getTransactionByBlockNumberAndIndex
* eth_getTransactionByHash
* eth_getTransactionCount
* eth_getTransactionReceipt
* eth_getUncleByBlockHashAndIndex
* eth_getUncleByBlockNumberAndIndex
* eth_getUncleCountByBlockHash
* eth_getUncleCountByBlockNumber
* eth_sendRawTransaction
* eth_getLogs ( not used in web3.js )

