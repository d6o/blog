+++
title = 'Using Go to Subscribe to Ethereum New Blocks and Print Transaction Hashes'
date = 2024-08-30T02:40:37-03:00
draft = false
tags = ["Go", "Ethereum", "Blockchain", "Programming", "Infura", "Web3", "Smart Contracts", "dApps", "Go-Ethereum", "Ethereum Development", "Blockchain Analytics", "Blockchain Monitoring", "Real-time Blockchain Data", "Ethereum Node", "Decentralized Applications"]
+++

In this post, we'll walk through how to use Go to subscribe to new blocks on the Ethereum blockchain and print out the
transaction hashes for each block. This is a common task when working with Ethereum, especially for monitoring or
analytics purposes.

## Prerequisites

Before we start, make sure you have the following:

1. **Go** installed on your machine.
2. Access to an Ethereum node. You can use a service like [Infura](https://infura.io/) or run your own Ethereum node.
3. The Go-Ethereum package (`go-ethereum`) installed. You can install it with:
   ```sh
   go get github.com/ethereum/go-ethereum
   ```

   ## Step 1: Setting Up the Project

First, create a new Go project and initialize it:

```sh
mkdir eth-subscribe
cd eth-subscribe
go mod init eth-subscribe
```

Next, we'll need to import the necessary packages. Create a `main.go` file and add the following imports:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"github.com/ethereum/go-ethereum"
	"github.com/ethereum/go-ethereum/rpc"
	"github.com/ethereum/go-ethereum/core/types"
)
```

## Step 2: Connecting to an Ethereum Node

To interact with the Ethereum blockchain, we need to connect to an Ethereum node. For this example, we'll use an Infura
endpoint, but you can replace the URL with your own node's address.

```go
func main() {
   // Replace with your Infura project ID or node address
   client, err := ethclient.Dial("wss://mainnet.infura.io/ws/v3/YOUR_INFURA_PROJECT_ID")
   if err != nil {
      log.Fatalf("Failed to connect to the Ethereum client: %v", err)
   }
   defer client.Close()

   // Create a new context
   ctx := context.Background()

   // Subscribe to new block headers
   headerCh := make(chan *types.Header)
   sub, err := client.SubscribeNewHead(ctx, headerCh)
   if err != nil {
      log.Fatalf("Failed to subscribe to new headers: %v", err)
   }

   for {
      select {
         case err := <-sub.Err():
            log.Fatalf("Subscription error: %v", err)
         case header := <-headerCh:
            processBlock(header, client)
      }
   }
}
```

## Step 3: Processing the Block

Once we have a new block header, we can use it to fetch the block and print out the transaction hashes:

```go
func processBlock(header *types.Header, client *ethclient.Client) {
   block, err := client.BlockByHash(context.Background(), header.Hash())
   if err != nil {
      log.Printf("Failed to get block: %v", err)
      return
   }

   fmt.Printf("New block #%d with %d transactions", block.Number().Uint64(), len(block.Transactions()))

   for _, tx := range block.Transactions() {
      fmt.Printf("Transaction hash: %s", tx.Hash().Hex())
   }
}
```

## Step 4: Running the Program

Finally, you can run your program to start subscribing to new blocks and printing out the transaction hashes:

```sh
go run main.go
```

As new blocks are mined on the Ethereum network, your program will output the block number along with the hashes of all
the transactions included in each block.

## Conclusion

In this tutorial, we have built a simple Go program that subscribes to new blocks on the Ethereum network and prints out
the transaction hashes. This is a foundational task for many blockchain applications, such as monitoring transactions,
building analytics tools, or creating decentralized applications (dApps).

Feel free to expand this example by filtering transactions, tracking specific contracts, or integrating with a database
to store the block data for further analysis.
