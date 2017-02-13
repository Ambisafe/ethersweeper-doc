<a href="https://www.ambisafe.co/" target="_blank">![test](img/logo_red.png)</a>
**********

# EtherSweeper
Daemon that forwards ETH deposits to ICAP addresses of <a href="https://github.com/Ambisafe/etoken-docs/wiki/eToken-API#transfertoicap" target="_blank">EToken API</a>

# Summary
EtherSweeper is a NodeJS package which operates a JSON-RPC API for ETH to ICAP forwarding. It allows the ETokenD user to manage incoming ETH through a WeiToken interface, and accumulate all the deposits in a single pool, distinguishing users by generated ICAP's.

**********

# Installation

**NodeJS 4.x or 6.x must be installed as a prerequisite.**
```
$ npm install
$ npm run-script build
```

# Running

Running **./bin/ethersweeper -h** will produce usage information.

```
$ ./bin/ethersweeper -h
usage: ethersweeper [-h] [-v] [--conf CONF] [--state STATE]
                    [--assetaddress ASSETADDRESS] [--secret SECRET]
                    [--confirmations CONFIRMATIONS] [--gas GAS]
                    [--gasprice GASPRICE] [--gethnode GETHNODE]
                    [--rpcbind RPCBIND] [--rpcport RPCPORT]
                    [--rpcuser RPCUSER] [--rpcpassword RPCPASSWORD]
                    [--logfile LOGFILE] [--unsafe UNSAFE]

EtherSweeper

Optional arguments:
  -h, --help            Show this help message and exit.
  -v, --version         Show program's version number and exit.
  --conf CONF           Specify configuration file (default: 
                        /etc/ethersweeper.conf)
  --state STATE         Json file to store the encrypted accounts. Backup 
                        regularly! (default: ethersweeper.json)
  --assetaddress ASSETADDRESS
                        Ethereum address of the WeiToken-like contract 
                        (default: 0x7660727d3cb947e807acead927ef3ede24c4a18d)
  --secret SECRET       Random string to be used as password for private keys 
                        encryption, mandatory
  --confirmations CONFIRMATIONS
                        After how many confirmations perform the forwarding 
                        transaction (default: 6)
  --gas GAS             What gas to use for forwarding (default: 500000)
  --gasprice GASPRICE   What gasPrice to use for forwarding (default: 
                        20000000000)
  --gethnode GETHNODE   geth node to talk to (default: https://localhost:8545)
                        Supernode service can be ordered at
                        https://www.ambisafe.co/services/
  --rpcbind RPCBIND     Bind to given address to listen for JSON-RPC 
                        connections (default: localhost)
  --rpcport RPCPORT     Listen for JSON-RPC connections on RPCPORT (default: 
                        28545)
  --rpcuser RPCUSER     Username for RPC basic auth (default: none)
  --rpcpassword RPCPASSWORD
                        Password for RPC basic auth (default: none)
  --logfile LOGFILE     Log file location
  --unsafe UNSAFE       Allow private keys export (default: false)
```

Running EtherSweeper with no command line args will result in an error due to absence of only mandatory parameter `--secret`, which should be known only to responsible admins and used to encrypt/decrypt local private keys storage. After providing that paramenter, application will start using WEI4 asset contract of EToken's test environment at 0x7660727d3cb947e807acead927ef3ede24c4a18d.

```
$ ./bin/ethersweeper --secret ChangeMe!
JSON-RPC server active on localhost:28545
```

To interact with EtherSweeper, you use the JSON-RPC API. For example:

```
$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"getinfo","params":[],"id":0}'
{"jsonrpc":"2.0","result":{"etherSweeper":true,"version":"1.0.0","routes":0,"forwards":0},"id":0}
```

The port used by EtherSweeper by default is the same as those used by geth, plus 20000.

# Config File

EtherSweeper can be configured entirely using the command line arguments, or it can read a config file which uses the same option names. The config file is specified with the **--conf** option, or read from */etc/ethersweeper.conf* by default. There are some example config files in the **bin** directory of the package.  If an option is set in the config file, as well as directly on the command line, the command line argument takes precedence.

# Basic Auth

EtherSweeper supports basic auth. The user and password can be set with the **rpcuser** and **rpcpassword** config file options, or the corresponding command line flags.

# Supported operations

```
$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"addroute","params":["XE43WE4ET00X1JW480KC"],"id":0}'
{"jsonrpc":"2.0","result":"0x9aCAeF26AeE7776d89B1F2d337cb0D92903Bc2b6","id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"editroute","params":["XE43WE4ET00X1JW480KC","XE45EXMAMBIZFKH4KFSA"],"id":0}'
{"jsonrpc":"2.0","result":"0x9aCAeF26AeE7776d89B1F2d337cb0D92903Bc2b6","id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"stoproute","params":["XE43WE4ET00X1JW480KC"],"id":0}'
{"jsonrpc":"2.0","result":"0x9aCAeF26AeE7776d89B1F2d337cb0D92903Bc2b6","id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"startroute","params":["XE43WE4ET00X1JW480KC"],"id":0}'
{"jsonrpc":"2.0","result":"0x9aCAeF26AeE7776d89B1F2d337cb0D92903Bc2b6","id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"getroute","params":["XE45EXMAMBIZFKH4KFSA"],"id":0}'
{"jsonrpc":"2.0","result":"0x9aCAeF26AeE7776d89B1F2d337cb0D92903Bc2b6","id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"listroutes","params":[],"id":0}'
{"jsonrpc":"2.0","result":{"XE45EXMAMBIZFKH4KFSA":{"addresses":["0x9acaef26aee7776d89b1f2d337cb0d92903bc2b6"]}},"id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"listforwards","params":[],"id":0}'
{"jsonrpc":"2.0","result":[{"icap":"XE43WE4ET00X1JW480KC","transactionHash":"0x6f44cd536bbf4dc25a1cb11be699e4a5969c5e7d45b3c7cd3de321ba8e905e6f"},{"icap":"XE43WE4ET00X1JW480KE","transactionHash":"0x5e353e991f970ff086f563e786b1d616c9f152d0a54eb901dc95fe58e2c15b29"}],"id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"listforwards","params":["XE43WE4ET00X1JW480KC"],"id":0}'
{"jsonrpc":"2.0","result":[{"icap":"XE43WE4ET00X1JW480KC","transactionHash":"0x6f44cd536bbf4dc25a1cb11be699e4a5969c5e7d45b3c7cd3de321ba8e905e6f"}],"id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"exportpk","params":["someaddress"],"id":0}'
{"jsonrpc":"2.0","result":"someprivatekey","id":0}

$ curl localhost:28545 -d '{"jsonrpc":"2.0","method":"shutdown","params":[],"id":0}'
{"jsonrpc":"2.0","result":"OK","id":0}
```

# Example of usage from NodeJS

```
var rpc = require('json-rpc2');
var client = rpc.Client.$create(18545, 'localhost');

client.call('addroute', ['XE43WE4ET00X1JW480KC'], function(err, result) { console.log(result); });
// console.log output: 0x9aCAeF26AeE7776d89B1F2d337cb0D92903Bc2b6
```
