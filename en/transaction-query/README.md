---
description: Tutorial on how to query transactions on the blockchain with Go.
---

# Querying Transactions

In the [previous section](../block-query) we learned how to read a block and all its data given the block number. We can read the transactions in a block by calling the `Transactions` method which returns a list of `Transaction` type. It's then trivial to iterate over the collection and retrieve any information regarding the transaction.

```go
for _, tx := range block.Transactions() {
  fmt.Println("Transaction information:")
  fmt.Println(tx.Hash().Hex())        // 0x7e7021d688f2e837f0c72726a12f0262cf5379b53d69c9ec71b98e83b2738b25
  fmt.Println(tx.Value().String())    // 0
  fmt.Println(tx.Gas())               // 1000000
  fmt.Println(tx.GasPrice().Uint64()) // 2500000020
  fmt.Println(tx.Nonce())             // 6
  fmt.Println(tx.Data())              // [ 166 209 253 255 0 ... 0 ]
  fmt.Println(tx.To().Hex())          // 0x39a4D265db942361D92e2B0039cae73Ea72a2ff9
}
```

In order to read the sender address, we need to call `AsMessage` on the transaction which returns a `Message` type containing a function to return the sender (from) address. The `AsMessage` method requires the EIP155 signer, which we derive the chain ID from the client.

```go
chainID, err := client.NetworkID(context.Background())
if err != nil {
	log.Fatal(err)
}

if msg, err := tx.AsMessage(types.NewLondonSigner(chainID), tx.GasPrice()); err == nil {
	fmt.Println(msg.From().Hex()) // 0xfCA5fAFf601f73C6FA9105BAc6B10B75F6595108
}
```

Each transaction has a receipt which contains the result of the execution of the transaction, such as any return values and logs, as well as the status which will be `1` (success) or `0` (fail).

```go
receipt, err := client.TransactionReceipt(context.Background(), tx.Hash())
if err != nil {
  log.Fatal(err)
}

fmt.Println(receipt.Status) // 1
fmt.Println(receipt.Logs) // ...
```

Another way to iterate over transaction without fetching the block is to call the client's `TransactionInBlock` method. This method accepts only the block hash and the index of the transaction within the block. You can call `TransactionCount` to know how many transactions there are in the block.

```go
blockHash := common.HexToHash("0x0cefcb3c3d24adaffd21cb68715d8cff093129568d857d3a52d3b851c59698c7")
count, err := client.TransactionCount(context.Background(), blockHash)
if err != nil {
  log.Fatal(err)
}

for idx := uint(0); idx < count; idx++ {
  tx, err := client.TransactionInBlock(context.Background(), blockHash, idx)
  if err != nil {
    log.Fatal(err)
  }

  fmt.Println(tx.Hash().Hex()) // 0x7e7021d688f2e837f0c72726a12f0262cf5379b53d69c9ec71b98e83b2738b25
}
```

You can also query for a single transaction directly given the transaction hash by using `TransactionByHash`.

```go
txHash := common.HexToHash("0x7e7021d688f2e837f0c72726a12f0262cf5379b53d69c9ec71b98e83b2738b25")
tx, isPending, err := client.TransactionByHash(context.Background(), txHash)
if err != nil {
  log.Fatal(err)
}

fmt.Println(tx.Hash().Hex()) // 0x7e7021d688f2e837f0c72726a12f0262cf5379b53d69c9ec71b98e83b2738b25
fmt.Println(isPending)       // false
```

---

### Full code

[transactions.go](https://github.com/miguelmota/ethereum-development-with-go-book/blob/master/code/transactions.go)

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/ethclient"
)

func main() {
	client, err := ethclient.Dial("https://mainnet.infura.io")
	if err != nil {
		log.Fatal(err)
	}

	blockNumber := big.NewInt(9930480)
	block, err := client.BlockByNumber(context.Background(), blockNumber)
	fmt.Println(block.Hash().Hex())
	if err != nil {
		log.Fatal(err)
	}
	for _, tx := range block.Transactions() {
		fmt.Println("Transaction information:")
		fmt.Println(tx.Hash().Hex())        // 0x7e7021d688f2e837f0c72726a12f0262cf5379b53d69c9ec71b98e83b2738b25
		fmt.Println(tx.Value().String())    // 0
		fmt.Println(tx.Gas())               // 1000000
		fmt.Println(tx.GasPrice().Uint64()) // 2500000020
		fmt.Println(tx.Nonce())             // 6
		fmt.Println(tx.Data())              // [ 166 209 253 255 0 ... 0 ]
		fmt.Println(tx.To().Hex())          // 0x39a4D265db942361D92e2B0039cae73Ea72a2ff9

		chainID, err := client.NetworkID(context.Background())
		if err != nil {
			log.Fatal(err)
		}

		if msg, err := tx.AsMessage(types.NewLondonSigner(chainID), tx.GasPrice()); err == nil {
			fmt.Println(msg.From().Hex()) // 0xfCA5fAFf601f73C6FA9105BAc6B10B75F6595108
		}

		receipt, err := client.TransactionReceipt(context.Background(), tx.Hash())
		if err != nil {
			log.Fatal(err)
		}

		fmt.Println(receipt.Status) // 1
	}

	blockHash := common.HexToHash("0x0cefcb3c3d24adaffd21cb68715d8cff093129568d857d3a52d3b851c59698c7")
	count, err := client.TransactionCount(context.Background(), blockHash)
	if err != nil {
		log.Fatal(err)
	}

	for idx := uint(0); idx < count; idx++ {
		tx, err := client.TransactionInBlock(context.Background(), blockHash, idx)
		if err != nil {
			log.Fatal(err)
		}

		fmt.Println(tx.Hash().Hex()) // 0x7e7021d688f2e837f0c72726a12f0262cf5379b53d69c9ec71b98e83b2738b25
	}

	txHash := common.HexToHash("0x7e7021d688f2e837f0c72726a12f0262cf5379b53d69c9ec71b98e83b2738b25")
	tx, isPending, err := client.TransactionByHash(context.Background(), txHash)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(tx.Hash().Hex()) // 0x7e7021d688f2e837f0c72726a12f0262cf5379b53d69c9ec71b98e83b2738b25
	fmt.Println(isPending)       // false
}
```
