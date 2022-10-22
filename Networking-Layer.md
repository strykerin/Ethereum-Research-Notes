# Notes

This repos contains my notes about my research on network protocols on Ethereum.

## Networking Layer

<https://ethereum.org/en/developers/docs/networking-layer/>

The networking layer consists of protocols that allow nodes to find each other and exchange information. There are 2 ways of communicating with other nodes: gossip (one-to-many) and request and response (one-to-one).

Each client (execution client and consensus client) has its own distinct networking stack. Execution clients gossip (one-to-many) transactions with other authenticated execution peers. Consensus clients gossip Beacon blocks accros their p2p network. This means there are two p2p networks:

- one network connecting execution clients for transaction gossip
- one network connecting consensus clients for block gossip
