# Notes

This repos contains my notes about my research on network protocols on Ethereum.

## Networking Layer

<https://ethereum.org/en/developers/docs/networking-layer/>

The networking layer consists of protocols that allow nodes to find each other and exchange information. There are 2 ways of communicating with other nodes: gossip (one-to-many) and request and response (one-to-one).

Each client (execution client and consensus client) has its own distinct networking stack. Execution clients gossip (one-to-many) transactions with other authenticated execution peers. Consensus clients gossip Beacon blocks accros their p2p network. This means there are two p2p networks:

- one network connecting execution clients for transaction gossip
- one network connecting consensus clients for block gossip

### Execution Client networking protocols

There are 2 stacks for this layer:

- **Discovery**: built on top of UDP. Allows a new node to find other peers
- **DevP2P**: build on top of TCP. Allows nodes to exchange information

#### Discovery (discv5)

When a new node joins the network, first it will contact the bootstrap nodes (hardcoded in the client implementation). The bootstrap nodes are only used when the node joins the network for the first time. After that, the node will no longer talk to them.

The protocol uses a modified form of Kademlia.

> Discovery starts with a game of PING-PONG. A successful PING-PONG "bonds" the new node to a bootnode. The initial message that alerts a bootnode to the existence of a new node entering the network is a PING. This PING includes hashed information about the new node, the bootnode and an expiry time-stamp. The bootnode receives the PING and returns a PONG containing the PING hash. If the PING and PONG hashes match then the connection between the new node and bootnode is verified and they are said to have `bonded`.
>
> Once bonded, the new node can send a FIND-NEIGHBOURS request to the bootnode. The data returned by the bootnode includes a list of peers that the new node can connect to. If the nodes are not bonded, the FIND-NEIGHBOURS request will fail, so the new node will not be able to enter the network.
>
> Once the new node receives a list of neightbours from the bootnode, it begins a PING-PONG exchange with each of them. Successful PING-PONG bond the new node with its neighbours, enabling message exchange.

```lang-none
start client --> connect to bootnode --> bond to bootnode --> find neighbours --> bond to neighbours
```
