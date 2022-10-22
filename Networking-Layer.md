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

##### Kademlia

<https://medium.com/coinmonks/a-brief-overview-of-kademlia-and-its-use-in-various-decentralized-platforms-da08a7f72b8f>

> Kademlia is a dht that provides a way for nodes to self-organize into a network and share resources between computers.
>
> On Kademlia each node has the mapping for a subset of the nodes on its own routing table.  This routing table is distributed among the nodes.

By having this distribute routes table, this allows the network to be resistant to dos attacks and loss of a group of nodes.

Kademlia defines the distance beween points in the key space by using the XOR. The distance between nodes is not the geographcial distance but instead the XOR of the id.

> For a network with 10,000,000 Kademlia nodes, only about 20 hops would be necessary at most for communication with any subset of nodes.

![Kademlia image](https://miro.medium.com/max/1100/1*VjRS6-nCY-QyCi5HmLfFaw.png)

Kademlia prefers long-lived nodes over newer entrants.

![Kademlia prob nodes uptime](https://miro.medium.com/max/1100/1*7teP36eX0zguNt8cdxahfQ.png)

> The process of joining a Kademlia network requires the discovery of only one peer, whereby the node then broadcasts its appearance. The node that recently joined the network then collects the NodeId from each response and adds it to its own dht.

Kademlia NodeId usually have 160 bits. The distance between 2 nodes is the XOR of the NodeIds.

> The structure of the Kademlia routing table is such that nodes maintain detailed knowledge of the address space close to them and exponentially decreasing knowledge of more distance address space.
>
> **K-buckets** are a list of routing addresses of other nodes in the network, which are maintained by each node and contains the IP address, port and NodeId for peer participants in the system. They prefer the longest-lived nodes, which means that one cannot overtake a node's routing state by flooding the system with new nodes (thus preventing certain types of ddos attacks).
>
> Kademlia requires that peers speak the same language so that they may find each other, recognize one another's position, and exchange messages.

The Kademlia protocol consists of four Remote Procedure Calls (RPCs):

- PING: Probes a node to see if it's online
- STORE: Instructs a node to store a key-value pair
- FIND_NODE: Returns information about the *k* closest to the target id
- FIND_VALUE: Similar to the FIND_NODE RPC, but if the recipient has received a STORE for a given key, it just returns the stored value

> Ethereum uses a slightly modified implementation of Kademlia. Ethereum utilizes the Kademlia's XOR metric and the k-bucket struct, and the lookup is mostly used to discover new peers.
> In Ethereum, the client stores information about other nodes in two data structures. The first is a long-term database, called db, which is stored on disk and persists across client reboots. The second is a short-term database, called table, which contains Kademlia-like buckets which are always empty whenever the client reboots.

##### dht

<https://en.wikipedia.org/wiki/Distributed_hash_table>

> The main advantage of a DHT is that nodes can be added or removed with minimum work around re-distributing keys. Keys are unique identifiers which map to particular values, which in turn can be anything, from addresses, to documents, to arbitrary data. Resposibility for maintaining the mapping from keys to values is distributed amon the nodes, is such a way that a change in the set of participants causes a minimal amount of disruption. This allows a dht to scale to extremely large number of nodes and to handle continual node arrivals, departure, and failues.

#### Ethereum Node Records (ENR)

<https://ethereum.org/en/developers/docs/networking-layer/#enr>
<https://ethereum.org/en/developers/docs/networking-layer/network-addresses/>
<https://github.com/ethereum/devp2p/blob/master/enr.md>

> The Ethereum Node Record is an object that contains three basic elements: a signature (hash of record contents made according to some agreed identity scheme), a sequence number that tracks changes to the record, and an arbitrary list of key:value pairs. This is a future-proof format that allows easier exchange of identifying information between new peers and is the preferred network address format for Ethereum nodes.
>
> A node record usually contains the network endpoints of a node: the node's IP addresses and ports. Node record also contains information about the node's purpose on the network.
>
> Ethereum nodes have to identify themselves with some basic information to connect to peers. To ensure any potential peer can interpret this information, it is relayed in one of three standardized formats that any Ethereum node can understand: multiaddr, enode, or Ethereum Node Records (ENRs). ENRs are the current standard for Ethereum network addresses.

### devp2p

[This repos](https://github.com/ethereum/devp2p) contains the specifications for devp2p

> devp2p is a set of network protocols which form the Ethereum peer-to-peer network. 'Ethereum network' is meant in a broad sense, i.e. devp2p isn't specific to a particular blockchain, but should serve the needs of any networked application associated with the Ethereum umbrella.

### discv5

<https://github.com/ethereum/devp2p/blob/master/discv5/discv5.md>

The node discovery protocol v5 is the protocol to find other nodes in the peer-to-peer network.
