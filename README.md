
# Peppermint-Chain: The Technical Architecture

The basic structure of Peppermint-Chain is that of an Operating-System. Like a regular Operating-System abstracts out the complexity of interacting with hardware into a simple set of APIs, Peppermint-Chain abstract out the complexity of building Blockchain Applications into a set of simple APIs based on proven technologies like Java & SQL. 

Peppermint-Chain like other OSes is internally composed of different modules which together provide all the functionality needed to create real-world Blockchain based applications. Modules such as

1. Data-Storage
2. Communication
3. Consensus & Smart-Contracts
4. Data-Integrity and Data-Confidentiality
5. Event Handling
6. Http-Server & Web-Services
7. Cryptographic Utilities

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

 
## Consensus & Smart-Contracts

