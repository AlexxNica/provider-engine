# Web3 ProviderEngine

Web3 ProviderEngine is a tool for composing your own [web3 providers](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3).

Status: WIP - expect breaking changes and strange behaviour

### Composable

Built to be modular - works via a stack of 'sub-providers' which are like normal web3 providers but only handle a subset of rpc methods.

The subproviders can emit new rpc requests in order to handle their own;  e.g. `eth_call` may trigger `eth_getAccountBalance`, `eth_getCode`, and others.
The provider engine also handles caching of rpc request results.

```js
const ProviderEngine = require('web3-provider-engine')
const CacheSubprovider = require('web3-provider-engine/subproviders/cache.js')
const StaticSubprovider = require('web3-provider-engine/subproviders/static.js')
const FilterSubprovider = require('web3-provider-engine/subproviders/filters.js')
const VmSubprovider = require('web3-provider-engine/subproviders/vm.js')
const LightWalletSubprovider = require('web3-provider-engine/subproviders/lightwallet.js')
const RpcSubprovider = require('web3-provider-engine/subproviders/rpc.js')

var engine = new ProviderEngine()
var web3 = new Web3(engine)

// cache layer
engine.addProvider(new CacheSubprovider())

// static results
engine.addProvider(new StaticSubprovider({
  web3_clientVersion: 'MetaMask-ProviderEngine/v0.0.0/javascript',
  net_listening: true,
  eth_hashrate: '0x0',
  eth_mining: false,
  eth_syncing: true,
})

// filters
engine.addProvider(new FilterSubprovider())

// vm
engine.addProvider(new VmSubprovider())

// id mgmt
engine.addProvider(new LightWalletSubprovider())

// data source
engine.addProvider(new RpcSubprovider({
  rpcUrl: 'https://testrpc.metamask.io/',
}))

// log new blocks
engine.on('block', function(block){
  // lazy hack - move caching and current block to engine
  engine.currentBlock = block
  console.log('================================')
  console.log('BLOCK CHANGED:', '#'+block.number.toString('hex'), '0x'+block.hash.toString('hex'))
  console.log('================================')
})

// start polling for blocks
engine.start()

```

### Built For Zero-Clients

The [Ethereum JSON RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC) was not designed to have one node service many clients.
However a smaller, lighter subset of the JSON RPC can be used to provide the blockchain data that an Ethereum 'zero-client' node would need to function.
We handle as many types of requests locally as possible, and just let data lookups fallback to some data source ( hosted rpc, blockchain api, etc ).
Categorically, we don’t want / can’t have the following types of RPC calls go to the network:
* id mgmt + tx signing (requires private data)
* filters (requires a stateful data api)
* vm (expensive, hard to scale)


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
- [ ] eth_newPendingTransactionFilter
- [x] eth_newFilter
- [x] eth_uninstallFilter
- [x] eth_getFilterLogs
- [x] eth_getFilterChanges

##### accounts manager
- [x] eth_coinbase
- [x] eth_accounts
- [x] eth_sendTransaction
- [ ] eth_sign ( not used in web3.js )

##### vm
- [x] eth_call
- [x] eth_estimateGas

##### db source
- [ ] db_putString
- [ ] db_getString
- [ ] db_putHex
- [ ] db_getHex

##### compiler
- [ ] eth_getCompilers
- [ ] eth_compileLLL
- [ ] eth_compileSerpent
- [ ] eth_compileSolidity

##### shh gateway
- [ ] shh_version
- [ ] shh_post
- [ ] shh_newIdentity
- [ ] shh_hasIdentity
- [ ] shh_newGroup
- [ ] shh_addToGroup

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

##### ( not supported )
* eth_getWork
* eth_submitWork
* eth_submitHashrate ( not used in web3.js )
