# dfuse for EOSIO
[![reference](https://img.shields.io/badge/godoc-reference-5272B4.svg?style=flat-square)](https://pkg.go.dev/github.com/dfuse-io/dfuse-eosio)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

All **[dfuse.io services](https://dfuse.io/technology)** for EOSIO,
running from your laptop or from a container, released as a single
statically linked binary: `dfuseeos`.

See the general [dfuse repository](https://github.com/dfuse-io/dfuse)
for other blockchain protocols implementations.

## Getting started

If it's the first time you boot a `nodeos` node, please review
https://developers.eos.io/welcome/latest/tutorials/bios-boot-sequence
and make sure you get a grasp of what this blockchain node is capable.

The default settings of `dfuseeos` allow you to quickly bootstrap a working
development chain by also managing the block producing node for you.

## Requirements

### Operating System
* Linux or macOS (no Windows support for now)

### dfuse Instrumented nodeos (deep-mind)
* See [DEPENDENCIES.md](DEPENDENCIES.md) for instructions on how to get an instrumented `nodeos` binary.

### Recommended tools
* [eosio.cdt tools](https://github.com/EOSIO/eosio.cdt)
* `cleos` (from [EOSIO/eos](https://github.com/EOSIO/eos)) or
* [eosc](https://github.com/eoscanada/eosc/releases).

## Installing

### From a pre-built release

* Download a tarball from the [GitHub Releases Tab](https://github.com/dfuse-io/dfuse-eosio/releases).
* Put the binary `dfuseeos` in your `PATH`.

### From source

Build requirements:
* `Git`
* `Go` 1.14 or higher ([installation](https://golang.org/doc/install#install))
* `yarn` 1.15 or higher ([installation](https://classic.yarnpkg.com/en/docs/install))
* [rice](https://github.com/GeertJohan/go.rice) Go static assets embedder (see installation instructions below)
* (On macOS) [Command Line Tools for Xcode](https://developer.apple.com/xcode/features/)

```
# Install `rice` CLI tool if you don't have it already
go get github.com/GeertJohan/go.rice
go get github.com/GeertJohan/go.rice/rice
```

_**Note** -- if you're getting yarn dependency warnings while running the `yarn install && yarn build` commands below, you can normally safely ignore those and move forward with the installation. If you're getting an error while installing and/or compiling, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md#installing--compiling-error)._

```
# clone the dfuse-eosio repo
git clone https://github.com/dfuse-io/dfuse-eosio
cd dfuse-eosio

pushd eosq
  yarn install && yarn build
popd

# To generate the dashboard box you will need to clone the dlauncher repo at the same level of dfuse-eosio
pushd ..
    git clone https://github.com/dfuse-io/dlauncher
    pushd dlauncher/dashboard
        pushd client
            yarn install && yarn build
        popd
        go generate .
    popd
popd

go generate ./dashboard
go generate ./eosq/app/eosq

go install -v ./cmd/dfuseeos
```

This will install the binary in your `$GOPATH/bin` folder (normally `$HOME/go/bin`). Make sure this folder is in your `PATH` env variable. If it's missing, take a look at [TROUBLESHOOTING.md](TROUBLESHOOTING.md#gopathbin-folder-missing-from-path-env-variable).


## Creating a new local chain with `dfuseeos`

### 1. Initialize
Initialize a few configuration files in your working directory (`dfuse.yaml`, `mindreader/config.ini`, ...)

```
dfuseeos init
```

_**macOS** -- If you get a prompt like `Do you want the application “dfuseeos” to accept incoming network connections?`, you should select `allow` as `dfuseeos` needs to accept connections from your local environment._


### 1a. Do you want dfuse for EOSIO to run a producing node for you
In this case, you want to create a new local chain so you should answer `y` (yes). The only time you would answer `n` (no) is if you intend to sync an existing chain with `dfuseeos`. If that's the case, see the [Syncing an existing chain with `dfuseeos`](#syncing-an-existing-chain-with-dfuseeos) section below. 

### 2. Boot

_**Note** -- see [booter/examples](./booter/examples) for other boot sequence templates._

Optionally, you can also copy over a boot sequence to have dfuse bootstraps your chain with accounts + system contracts to have a chain ready for development in a matter of seconds:

```
wget -O bootseq.yaml https://raw.githubusercontent.com/dfuse-io/dfuse-eosio/develop/booter/examples/bootseq.dev.yaml
```

When you're ready, boot your instance with:

```
dfuseeos start
```

A successful start will list the launching applications as well as the graphical interfaces with their relevant links:

```
Dashboard:        http://localhost:8081
Explorer & APIs:  http://localhost:8080
GraphiQL:         http://localhost:8080/graphiql
```

In this mode, two nodeos instances will now be running on your machine, a block producer node and a mindreader node, and the dfuse services should be ready in a couple seconds.

## Syncing an existing chain with `dfuseeos`

If you chose to sync to an existing chain, only the mindreader node will launch. It may take a while for the initial sync depending on the size of the chain and the services may generate various error logs until it catches up (more options for quickly syncing with an existing chain will be proposed in upcoming releases).

* See [Syncing a chain partially](./PARTIAL_SYNC.md)
* See the following issue about the complexity of [syncing a large chain](https://github.com/dfuse-io/dfuse-eosio/issues/26)

You should also take a look at our Docs:
* [System Admin Guide](https://docs.dfuse.io/eosio/admin-guide/)
* [Large Chains Preparation](https://docs.dfuse.io/eosio/admin-guide/large-chains-preparation/)

## Overview - Repository Map

The glue:
* The [dfuseeos](./cmd/dfuseeos) binary.
* The [launcher](./launcher) which starts all the internal services

The EOSIO-specific services:
* [abicodec](./abicodec): ABI encoding and decoding service
* [fluxdb](./fluxdb): the **dfuse State** database for EOSIO, with all tables at any block height
* [kvdb-loader](./kvdb-loader): service that loads data into the `kvdb` storage
* [dashboard](./dashboard): server and UI for the **dfuse for EOSIO** dashboard.
* [eosq](./eosq): the famous https://eosq.app block explorer
* [eosws](./eosws): the REST, Websocket service, push guarantee, chain pass-through service.

dfuse Products's EOSIO-specific hooks and plugins:
* [search plugin](./search), object mappers, EOSIO-specific indexer, results mapper (along with the [search client](./search-client).
* [dgraphql resolvers](./dgraphql), with all data schemas for EOSIO
* [blockmeta plugin](./blockmeta), for EOS-specific `kvdb` bridge.

## Logging

See [Logging](./LOGGING.md)

## Troubleshooting

See [Troubleshooting](./TROUBLESHOOTING.md)

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our Code of Conduct, [CONVENTIONS.md](CONVENTIONS.md) for coding conventions, and processes for submitting pull requests.

## License

[Apache 2.0](LICENSE)

## References

- [dfuse Docs](https://docs.dfuse.io)
- [dfuse on Telegram](https://t.me/dfuseAPI) - Community & Team Support
