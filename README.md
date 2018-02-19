
# Peppermint-Chain: The Technical Architecture

The basic structure of Peppermint-Chain is that of an Operating-System. Like a regular Operating-System abstracts out the complexity of interacting with hardware into a simple set of APIs, Peppermint-Chain abstract out the complexity of building Blockchain Applications into a set of simple APIs based on proven technologies like Java & SQL. 

Peppermint-Chain like other OSes is internally composed of different modules which together provide all the functionality needed to create real-world Blockchain based applications. Modules such as

1. Data-Storage
2. Communication
3. Consensus & Smart-Contracts
4. Event Handling
5. Http-Server & Web-Services
6. Cryptographic Utilities

Applications on Peppermint-Chain are packages that have a combination of code and declarations that are registered with the different internal modules of the System and interact with other Applications and the Network to acheive their given objectives.

## Data-Storage Module
The **Data-Storage** module is responsible for the long term persistent storage of Application data and the permissions structure to allow multiple Applications to share persistent data.

In Peppermint-Chain all persistent data is stored in a Relational Data Model. As such each Application defines its Data-Model (in terms of *Tables,Views and Indexes*) that it will be using to store the data. Both the retrieval and modifications of the data can be accomplised using ANSI standard SQL.

Internally the data is stored in a Temporal database to allow access to the state of the system at any point in time. This is required since in a Blockchain the state evolves over time and based on the forking of blocks might require time before it can be considered committed.

The Data-Model of every Application consists of both a Consensus Part and a Private part. The Consensus Part of the Data-Model can only be modified by *Smart-Contracts* during the Consensus phases of a Transaction. The private part of the Data-Model is modifiable by the UI and Event-Handlers. 

Every Application defines its own Data-Model and the parts of other Application Data-Models it intends to access/modify. At Application installation time this information is confirmed by the Installing User. The **Data-Storage** sub-system tracks these permissions and enforces them.

## Communication Module
Every Blockchain Application derives most of its value from interaction with other Nodes in a Blockchain and the consensus they reach. The *Communication* module deals with all the interaction between Nodes. This includes communications for

1. Node Discovery and Registration
2. Blockchain based Consensus
3. Data Transfer
4. Data Integrity Verification

The Communication Module takes care of the basic of packets and messages, allowing the Application code itself to work more at an abstract level dealing with the Application Logic.

Currently the Communication Module uses Google-Protobuf messages over a TCP/IP but we are thinking of adding HTTP as a transport to allow tunnelling through Firewalls if needed.

 
## Smart-Contracts & Consensus


## Event Handling

## HTTP-Server & Web-Services

## Cryptographic Utilities

# Appendix-A: Peppermint-Consensus

When a *Smart-Contract* transaction needs to be executed, the **Client** connects to the relevant nodes and first simulates the transaction.
To invoke the *Simulation* the **Client** passes the following
1. The *Smart-Contract* to invoke.
2. A *Transaction Key*. This is a randomly generated AES key.
3. The input arguments for the *Smart-Contract*.
 
In the *Simulation Stage* each node executes the *Smart-Contract* associated with the transaction passing it the data sent from the client.
The *Smart-Contract* is a regular java class which is passed the input data and a **JDBC Connection** to the state of the system. The *JDBC Connection* is designed to only allow access to the tables defined at the Application level.
The *Smart-Contract* uses the input data to access/modify the node state using regular **JDBC** syntax and *ANSI SQL*.
The node tracks the rows of data that have been accessed/modified but does not commit the changes. Instead it creates a *Merkle Tree* of all the changes and sends a **Simulation Message* to the **Client**.
In addition the node will lock all Row Keys modified by the *Transaction*. These locks will be held till either the *Commit Block Number* or the **Simulation** is committed as explained below. 

The **Simulation Message** is signed by the node's *Private Key* and consists of 
1. The Merkle-Tree root hash.
2. A Commit By Block Number - This is typically the Latest Block No known to the Node + a pre-configured Delta.

This **Simulation Message** guarantees the client that if a **Commit Message** for the Transaction is seen on the Blockchain on or before the specified *Commit By Block Number*, the data created by the Simulation will be committed to the state. 

The **Commit Message** consists 
1. A Minimum Block No. This is the minimum of the all the *Commit By Block Numbers* received in the **Simulation Messages** from the Nodes.
2. The Merkle-Tree root hash.
3. Signed **Simulation Messages** from all the relevant parties, encrypted by the **Transaction Key**.
 

The **Client** on its part will get multiple commitment messages from the nodes. It compares the Merkle roots sent from all the nodes and if and only if all of them match it creates a **Commit Message** and submits it to the Blockchain for inclusion in a block with the condition that it needs to be included in a block which has a block no less than equal to the **Minimum Block No** on the message.

If the **Commit Message** is added to the Blockchain then all the nodes receive it and will commit the data and release any locks.

So the final state of a committed Transaction is
1. Every node has a signed message from all the other nodes acknowledging the Merkle-Tree root hash. 
2. The Transaction data matching the Merkle-Tree root hash.
3. The Blockchain containing the Merkle-Tree root hash.

Since the hashes match it can be assumed that all the nodes have consensus on the data of the Transaction. 

Peppermint-Chain's Transaction can be viewed as Two-Phase commit transaction across multiple nodes. The initial *Simulation* in Peppermint-Chain is the equivalent of the *Commit Request Phase* of a Two-Phase Commit. And the *Commit Message* in the Blockchain is the equivalent of the *Commit Phase* of a Two-Phase Commit.

### Failure Scenarios

#### Merkle-Root hashes do not match
The **Client** reports to the user that there was no **Consensus** and aborts the Transaction. Each node has a **Simulation** with a **Commit By Block Number**. When the Merkle-Root hash is not seen on the Blockchain by the *Commit By Block Number*, the node will discard the **Simulation** and any associated locks with it.

#### Merkle-Root hashes match but the "Commit Message" is missing
The most likely scenario for this is that the Commit Message was broadcast by the Client but could not make it into the Blockchain in time. The **Client** will at this point re-attempt the transactions. We are working on some features to allow Clients to specify a longer interval. The design will be published after review.

### Contention Scenarios

There might be a second Transaction trying to modify a row which has already been locked by a previous **Simulation**. In this case Peppermint-Chain will abort the Transaction and re-attempt it once the original Transaction's locks have been released.



