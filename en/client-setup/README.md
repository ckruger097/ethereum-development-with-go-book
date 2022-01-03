---
description: Tutorial on how to set up a client to connect to Ethereum with Go.
---

# Setting up the Client

Setting up the Ethereum client in Go is a fundamental step required for interacting with the blockchain. First import the `ethclient` go-ethereum package and initialize it by calling `Dial` which accepts a provider URL.

You can connect to the infura gateway if you don't have an existing client. Infura manages a bunch of Ethereum [geth and parity] nodes that are secure, reliable, scalable and lowers the barrier to entry for newcomers when it comes to plugging into the Ethereum network.

```go
client, err := ethclient.Dial("https://mainnet.infura.io")
```

You may also pass the path to the IPC endpoint file if you have a local instance of geth running.

```go
client, err := ethclient.Dial("/home/user/.ethereum/geth.ipc")
```

Using the ethclient is a necessary thing you'll need to start with for every Go Ethereum project and you'll be seeing this step a lot throughout this book.

## Using Ganache

[Ganache](https://github.com/trufflesuite/ganache-cli) (formally known as *testrpc*) is an Ethereum implementation written in Node.js meant for testing purposes while developing dapps locally. Here we'll walk you through how to install it and connect to it.

First install ganache via [NPM](https://www.npmjs.com/package/ganache-cli).

```bash
npm install -g ganache-cli
```

Then run the ganache CLI client.

```bash
ganache-cli
```

Now connect to the ganache RPC host on `http://localhost:8545`.

```go
client, err := ethclient.Dial("http://localhost:8545")
if err != nil {
  log.Fatal(err)
}
```

You may also use the same mnemonic when starting ganache to generate the same sequence of public addresses.

```bash
ganache-cli -m "much repair shock carbon improve miss forget sock include bullet interest solution"
```

I highly recommend getting familiar with ganache by reading their [documentation](http://truffleframework.com/ganache/).

For these exercises, you may choose to either work on a local network (using ganache-cli) or through a free testnet gateway such as Infura. The difference between the two is that a testnet (such as Rinkeby or Kovan) is much closer to the actual Ethereum mainnet, as its transactions are mined by actual miners, thus requiring testnet ETH to use. This is not "real" ETH and does not cost money, instead these test networks use "testnet" ETH that is available at various
[faucets](https://faucet.rinkeby.io/). These faucets deposit a small amount (up to 1 ETH/day) of testnet Ether and are not always available at any given time. One advantage to using testnets is that other individuals or organizations outside your local network can interact with your smart contracts/dApps in the same way as they would on the Ethereum mainnet. 
---

### Full code

[client.go](https://github.com/miguelmota/ethereum-development-with-go-book/blob/master/code/client.go)

```go
package main

import (
	"fmt"
	"log"

	"github.com/ethereum/go-ethereum/ethclient"
)

func main() {
	client, err := ethclient.Dial("https://mainnet.infura.io")
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("we have a connection")
	_ = client // we'll use this in the upcoming sections
}
```
