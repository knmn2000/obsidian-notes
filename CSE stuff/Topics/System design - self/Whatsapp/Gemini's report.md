# Deconstructing WhatsApp: An Expert Report on the Architecture of a Global Messaging Behemoth

## Introduction

This report provides a definitive technical explanation of WhatsApp's real-world backend system. It moves beyond the theoretical designs common in system design interviews to analyze the specific, and often unconventional, engineering decisions that enable its unprecedented scale and reliability. WhatsApp's architecture is a masterclass in purpose-built engineering, where a maniacal focus on vertical scalability, enabled by a homogenous Erlang-based technology stack and deep, low-level performance tuning, created one of the most efficient and resilient communication platforms in the world.

The analysis will dissect the core technology stack, the high-level architecture, and the mechanics of its critical services, including the messaging pipeline, presence system, group chat functionality, and media handling. Furthermore, it will investigate the platform's robust security posture, its unique performance optimization strategies, and its recent, fundamental architectural evolution to support a true multi-device experience.

## The Bedrock: WhatsApp's Core Technology Stack and Engineering Philosophy

The foundation of WhatsApp's technical prowess lies in a small, tightly integrated set of technologies, chosen not for their popularity, but for their specific suitability to the unique challenges of a massive, real-time messaging system. This stack reflects an engineering philosophy that prizes simplicity, operational efficiency, and extreme performance.

### The Erlang/OTP Advantage: Concurrency, Fault Tolerance, and Hot-Swapping

The single most important technology choice made by WhatsApp was the adoption of the Erlang programming language and its accompanying Open Telecom Platform (OTP) framework.1 Originally developed at Ericsson in 1986 for building highly reliable telephony switches, Erlang is designed with features that map almost perfectly to the requirements of a global messaging service.3

- **Lightweight Processes and the Actor Model:** Erlang's concurrency is based on the "Actor Model," where computation is performed by extremely lightweight, isolated processes that communicate by passing asynchronous messages.2 These are not heavyweight operating system threads; the Erlang virtual machine can manage hundreds of thousands, or even millions, of these processes on a single server.3 This allows WhatsApp to architect its system around a "one process per user connection" model, dramatically simplifying state management compared to traditional architectures that rely on complex thread pools and external session stores.5
    
- **Fault Isolation and Self-Healing Systems:** A cornerstone of Erlang's design is the "let it crash" philosophy.8 Processes are completely isolated; a crash or error in one process has no effect on any other process in the system.1 OTP provides "supervisors," which are special processes whose only job is to monitor other processes and restart them if they fail. This enables the creation of self-healing, highly fault-tolerant systems, a critical attribute for a service that must maintain constant availability for billions of users.2
    
- **Hot Code Swapping:** Erlang was designed for systems that could never be taken down for maintenance, like telephone exchanges. To facilitate this, it supports "hot code swapping," the ability to load and activate new application code in a running production system without stopping or restarting the service.7 This feature allows for continuous deployment, rapid bug fixes, and extremely long server uptimes, measured in months or even years.7
    

### BEAM: The High-Concurrency Virtual Machine

Erlang code is compiled into bytecode that runs on the BEAM (Bogdan's Erlang Abstract Machine) virtual machine.2 The BEAM is the engine that brings Erlang's theoretical advantages to life. It is purpose-built for managing massive numbers of concurrent, I/O-bound tasks.1

The BEAM's preemptive schedulers are a key component of its performance. Each CPU core is typically assigned a scheduler, which manages the execution of thousands of Erlang processes. The schedulers ensure that no single process can monopolize a core, providing the soft real-time responsiveness necessary for a low-latency messaging application.2 This design provides near-linear Symmetric Multiprocessing (SMP) scalability, allowing WhatsApp to effectively utilize the resources of very large, multi-core servers and sustain CPU utilization above 85% under heavy production loads.11 The WhatsApp engineering team has been so deeply involved with the VM that they have contributed numerous patches and fixes directly to the BEAM source code to tune it for their specific workload, a testament to their full-stack optimization approach.2

### FreeBSD: The Optimized Operating System for a High-Throughput Network Stack

In a move that bucked industry trends, WhatsApp's founders chose FreeBSD as their operating system over the more ubiquitous Linux.1 Co-founder Brian Acton cited FreeBSD's relative simplicity and stability, calling Linux a "beast of complexity" and praising FreeBSD's "extraordinarily good ports collection".2 Their extensive prior experience with FreeBSD at Yahoo! was also a decisive factor.2

From a performance perspective, FreeBSD is renowned for its highly optimized and robust networking stack, which delivers exceptional performance in terms of system load per packet—a critical metric for a service whose primary function is to route billions of small data packets daily.2 The WhatsApp team did not use FreeBSD off-the-shelf; they engaged in deep kernel tuning, increasing the system-wide limits for file descriptors and sockets, and even backported drivers and features from newer FreeBSD versions to their production environment to resolve specific performance bottlenecks.11

### Mnesia and ETS: The Real-Time Distributed Database System

For its database needs, WhatsApp again turned to the Erlang ecosystem, selecting Mnesia. Mnesia is a distributed, soft real-time Database Management System (DBMS) that is written in Erlang and is a standard component of the OTP framework.1 It offers features like real-time key/value lookups, high fault tolerance through replication, and dynamic schema reconfiguration.2

A key advantage of Mnesia is its tight integration with the Erlang language itself. There is no data structure or "impedance mismatch" between the application code and the database; Erlang terms can be stored directly in Mnesia tables, which simplifies development and eliminates the overhead of data serialization and deserialization layers common in other architectures.2 WhatsApp primarily uses Mnesia for storing transient or semi-permanent data, most notably offline messages waiting for a user to reconnect.2 Once a message is successfully delivered, it is wiped from the server's Mnesia store.2 It is also used for storing user account metadata and group information.1

For data requiring even lower latency and no disk persistence, the system uses Erlang Term Storage (ETS). ETS provides in-memory, non-persistent tables that offer extremely fast, concurrent access. It is ideal for caching frequently accessed data or managing ephemeral state, such as the mapping of users to server processes or controlling lock contention on shared resources.8

### The Legacy of XMPP: Customizing Ejabberd into FunXMPP

WhatsApp's initial messaging protocol was built upon the foundation of XMPP (Extensible Messaging and Presence Protocol), an open standard for real-time communication.2 They started with

`ejabberd`, a popular and robust open-source XMPP server written in Erlang, a natural choice given the rest of their stack.2

However, standard XMPP, which is based on verbose XML stanzas, was ill-suited for the high-latency, low-bandwidth environment of mobile networks. To overcome this, WhatsApp heavily customized both the protocol and the server, creating a proprietary, binary-efficient version known as FunXMPP.18 This involved stripping down the protocol to its essentials and implementing a binary encoding scheme to drastically reduce the size of data transmitted over the network.20 While the modern protocol has evolved significantly, its architecture retains the core principles of XMPP—such as presence notifications and message stanzas—but implemented over a custom binary protocol on a persistent TCP or WebSocket connection.1

The selection of these individual technologies reveals a broader, unifying strategy: the creation of a homogenous development ecosystem. The choice of Erlang naturally led to the adoption of `ejabberd`, Mnesia, and the web server YAWS, all components of the same ecosystem. This created a highly integrated environment where developers could become deep experts in a single, cohesive platform. Optimizations made to the underlying BEAM virtual machine would benefit the entire stack simultaneously—from the database to the application logic to the web-serving layer. This approach acted as a powerful force multiplier, allowing a famously small engineering team to build and manage a system of immense scale and complexity.7

This technology stack is also a direct consequence of a deeply ingrained engineering culture of "Just Enough Engineering".7 With a lean team, simplicity is not merely a preference but a crucial survival trait. The focus on a unified stack reduced the cognitive and operational overhead that would have come from a more heterogeneous mix of technologies. By ruthlessly avoiding feature creep and concentrating on the core messaging function, the team kept the system manageable, maintainable, and scalable.7 The technology choices did not just enable the product; they were a physical manifestation of the minimalist and pragmatic culture that defined WhatsApp's engineering success.

|Component|Technology|Primary Rationale for Choice|
|---|---|---|
|**Programming Language**|Erlang|Extreme concurrency, built-in fault tolerance ("let it crash"), and hot code swapping for high availability.1|
|**Virtual Machine**|BEAM|High-performance SMP scheduling for near-linear scalability on multi-core CPUs; management of millions of lightweight processes.1|
|**Operating System**|FreeBSD|High-performance and stable networking stack, simplicity of distribution, and deep familiarity within the founding engineering team.2|
|**Core Database**|Mnesia / ETS|Real-time, in-memory/distributed key-value storage; zero impedance mismatch with Erlang application code; used for transient data like offline messages.2|
|**Web Server**|YAWS|An Erlang-based web server used for handling HTTP-based traffic, primarily for media uploads and downloads, maintaining stack homogeneity.2|
|**Messaging Protocol**|Custom Binary Protocol|Evolved from a heavily modified version of XMPP (`ejabberd`) into a proprietary binary protocol (FunXMPP) optimized for low-latency and efficiency on mobile networks.2|

## High-Level System Architecture: A Blueprint for Global Scale

WhatsApp's high-level architecture is designed for massive horizontal scalability and extreme resilience. It eschews many conventions of modern web-scale architectures in favor of a stateful, vertically-scaled model optimized for its unique workload.

### Architectural Overview: From a Centralized Core to Partitioned "Islands"

The system is fundamentally a distributed network of Erlang nodes operating on bare-metal servers, historically hosted by SoftLayer (now IBM Cloud) across multiple data centers.1 While it follows a client-server pattern, the backend is not a single monolithic application. Instead, it is broken down into over 40 distinct functional clusters, each dedicated to a specific task such as chat session management, multimedia message (MMS) handling, user authentication, or spam detection.8 This logical decoupling is a key design principle, as it isolates failures. An issue within the media upload cluster, for example, will not cascade to affect the core text messaging service, thereby containing the blast radius of any potential problem.8

To further enhance fault tolerance, these backend nodes are organized into "islands of stability".8 An island is a self-contained unit, typically consisting of a primary and a secondary node, responsible for a specific partition of the user base or data. Data for a given partition is replicated from the primary to the secondary node. If the primary node fails, the secondary can take over its responsibilities almost instantaneously. This model ensures that most failures are confined to a single island, preventing catastrophic system-wide outages and providing a high degree of availability without the overhead of full data replication across the entire global backend.8

### The Connection Layer: Managing Millions of Persistent Sessions

Unlike standard web applications that rely on stateless HTTP requests, WhatsApp's core functionality is built upon long-lived, stateful TCP connections maintained between the client and the server.3 For web and desktop clients, this is typically implemented using WebSockets to facilitate persistent, bidirectional communication.14

The architectural decision to assign a dedicated, lightweight Erlang process on the server to manage each client connection is central to the system's design.8 This "one process per connection" model allows the server to maintain all session-related state—such as the user's online status, connection information, and transient message queue—directly within the memory of that process. This co-location of state and logic is highly efficient, eliminating the network latency and complexity associated with querying an external session store (like Redis) for every incoming message. When a user disconnects, the corresponding Erlang process terminates, and its resources are garbage-collected, ensuring clean state management.8

This stateful architecture is a direct enabler of WhatsApp's philosophy of vertical scaling. The explicit goal was to maximize the number of concurrent connections handled by a single server, with early targets of 1 million connections per server later being pushed to over 2 million.12 Achieving this density required the unparalleled process-handling efficiency of the Erlang BEAM VM combined with the extensive, low-level tuning of the FreeBSD kernel, as detailed later in this report.11 This approach reflects a conscious trade-off: prioritizing the reduction of operational complexity by managing fewer, more powerful servers, even at the cost of making each individual server a highly specialized and complex piece of engineering. The guiding principle is that operational complexity scales with the number of nodes, not the number of cores within them.12

### Service Discovery and Routing

A client connecting to WhatsApp does not simply hit a generic load balancer that assigns it to a random server. The process is more structured to support the stateful nature of the system.

1. **Chat Registry:** Upon initial connection, the client contacts a dedicated service, referred to as a "Chat Registry".27 This service acts as a directory. Based on the user's ID and the current load across the server fleet, the registry assigns the client to a specific chat server and provides the client with its address. The client then establishes its persistent connection directly with that assigned server.
    
2. **Inter-Server Routing:** Once connected, a user on Chat Server A may need to send a message to a user whose session is active on Chat Server B. For this to work, Server A must be able to discover the location of the recipient's session process. This inter-node routing information is managed through a combination of mechanisms. At a high level, a distributed coordination service like Zookeeper or an internal equivalent can be used to maintain the mapping of which users are on which servers.27 Within the Erlang ecosystem itself, WhatsApp makes extensive use of
    
    `pg2`, a built-in distributed process group manager, which allows processes to join named groups (e.g., a group for a specific user ID partition) and enables efficient message broadcasting and routing to those groups across the cluster.12 For communication that must span across different data centers, the team built a custom Erlang distribution transport protocol called
    
    `wandist` that operates over standard `gen_tcp`.12
    

This embrace of statefulness at the connection tier is a significant departure from the stateless application tiers favored by many modern web-scale systems. Where a stateless architecture would necessitate an external cache lookup for every message to retrieve routing information, adding latency and dependencies, WhatsApp's model leverages the built-in capabilities of the distributed Erlang runtime. A message arriving at a sender's process can be routed directly to the recipient's process mailbox, even if it resides on a different physical machine, with the state (the socket connection) co-located with the logic (the process). This design trades the scaling simplicity of stateless nodes for the raw performance and logical simplicity of stateful message passing—a trade-off that is highly advantageous for a real-time system where minimizing latency is the paramount concern.

## Deep Dive: The Mechanics of Core Services

The high-level architecture is brought to life by a set of specialized services, each optimized for its specific function. The design of these services reflects a consistent theme: keeping the core real-time path as lean and fast as possible by offloading heavy or non-essential work to decoupled subsystems.

### The Messaging Pipeline

The flow of a message through WhatsApp's backend differs significantly depending on the online status of the recipient.

#### Real-Time Delivery (Online Users)

When both sender and receiver are online, the message delivery path is optimized for minimal latency.

1. User A's client sends a message. The message travels over the persistent connection to the dedicated Erlang process managing User A's session on their assigned chat server, let's call it Chat Server 1.28
    
2. Upon receiving the message, Chat Server 1 immediately sends an acknowledgment back to User A's client. This acknowledgment triggers the "single checkmark" status on the sender's device, confirming server receipt.28
    
3. Chat Server 1 then performs a lookup using the distributed service registry (`pg2` or a similar mechanism) to find the location of the Erlang process for User B.12 Let's assume User B is connected to Chat Server 2.
    
4. The message is forwarded directly from the process on Server 1 to the process on Server 2 using Erlang's efficient inter-node communication.
    
5. The process on Server 2 pushes the message down the persistent connection to User B's device.
    
6. User B's client, upon receiving the message, sends a "delivered" acknowledgment back to Server 2. This status is relayed back to Server 1 and then to User A's client, triggering the "double checkmark" status.2
    

It is crucial to note that the server's role here is purely that of a router. The ultimate source of truth for a user's chat history is their own device, where conversations are stored in a local SQLite database.2

#### Offline Message Handling (Transient Storage)

When the recipient is offline, their session process does not exist on any chat server. The delivery flow changes to accommodate this.

1. The initial steps are the same: User A sends the message to Chat Server 1.
    
2. Server 1 attempts to look up User B's session process but finds that they are not connected.
    
3. Instead of attempting delivery, the message is handed off to a specialized "Transient Service".28
    
4. This service persists the message in a temporary storage queue. The primary technology used for this is the Mnesia database, which is optimized for the high-throughput, "dirty" asynchronous writes required for this task.2
    
5. When User B next comes online, their client establishes a new session with an assigned chat server. As part of the connection process, the server (or the client itself) queries the Transient Service and retrieves all pending messages queued for User B.2
    
6. These messages are delivered to User B's client. Once the client acknowledges successful receipt of each message, the corresponding entries are permanently deleted from the Mnesia database on the server. The server does not retain message history long-term, reinforcing the principle of data minimization.2
    

#### Message Status: Sent, Delivered, Read Receipts

The checkmark system provides users with real-time feedback on message state.

- **Sent (Single Gray Checkmark):** This status appears the moment the sender's client receives an acknowledgment from its connected chat server that the message has been successfully received and queued for delivery.28
    
- **Delivered (Double Gray Checkmark):** This status is triggered when the recipient's client receives the message and sends its own acknowledgment back to the server, which is then forwarded to the sender.2 This confirms the message has reached the recipient's device but not necessarily that it has been seen.
    
- **Read (Double Blue Checkmark):** This is a purely application-level signal. When the recipient opens the specific chat thread and the message becomes visible on their screen, the client application sends a "read" receipt to the server. This receipt is then relayed to the sender's client, which updates the UI accordingly.3
    

### The Presence System ("Online" / "Last Seen")

The ability to see if a contact is "online" or their "last seen" time is a core feature that relies on a classic publish-subscribe (pub-sub) architecture.

- **Mechanism:** When a user's WhatsApp application is active and in the foreground, it maintains its persistent connection to the server. It may also send periodic heartbeat signals to explicitly confirm its active presence.3 A user's status—be it "online," "offline," or "typing..."—can be thought of as a topic in a pub-sub system.
    
- **Architecture:**
    
    1. **Publish:** When a user's status changes (e.g., they open the app), their client "publishes" this event to a dedicated Presence Service.29
        
    2. **Subscribe:** The contacts of that user who are currently online are effectively "subscribers" to that user's presence topic.
        
    3. **Fan-out:** The Presence Service receives the status update and fans it out to all relevant, online subscribers, who then update their UI to show the user as "online."
        
- **Technology:** While generic system designs often propose using a tool like Redis for managing presence sets 14, it is highly probable that WhatsApp leverages Erlang's native capabilities for this. The built-in
    
    `pg2` module provides a powerful and efficient mechanism for managing distributed process groups, which is an ideal foundation for implementing pub-sub logic directly within the BEAM ecosystem without introducing external dependencies.12 The "last seen" timestamp is simply the last recorded time of a user's activity, which is updated when their client gracefully disconnects or when their presence heartbeat times out after a configured interval.29
    

### Group Messaging Architecture

Delivering a single message to potentially hundreds of recipients efficiently and securely presents a unique architectural challenge. The solution has evolved over time, reflecting a classic trade-off between performance and the strengthening of the security model.

#### The "Sender Keys" Protocol and Server-Side Fan-Out

Initially, WhatsApp's group chat encryption was built on a component of the Signal Protocol called "Sender Keys," designed for efficient one-to-many encrypted communication.36

- **Key Distribution:** The first time a member sends a message to a new group, their client generates a random, long-lived group encryption key (the "Sender Key"). It then encrypts this Sender Key once for every other member of the group, using the secure one-on-one session already established with each of them. These individually encrypted Sender Keys are then distributed to each member.
    
- **Message Sending:** From that point forward, when that member sends a message to the group, they encrypt it just once using their Sender Key. They then transmit this single ciphertext to the WhatsApp server.
    
- **Server-Side Fan-Out:** The server, which cannot decrypt the message, performs a "fan-out." It looks up the membership list for the group and delivers a copy of the single encrypted message to every member.36 This model is extremely efficient for the sending client, which only needs to perform one encryption and one upload, minimizing bandwidth and battery usage. Group membership itself is managed by the server.39
    

#### Evolution to Client-Fanout with Multi-Device

The introduction of true multi-device support, where each user can have multiple independent devices, necessitated a change in this model. The modern architecture employs a **client-fanout** approach.40 In this model, the sending client is responsible for encrypting the message multiple times—once for

_each individual device_ of every recipient in the group—and transmitting all of these encrypted copies to the server. This shifts the fan-out work from the server to the client. While this increases the workload on the sending device, it strengthens the end-to-end encryption model by removing the server's role in managing group message distribution and aligns better with a decentralized identity model.

### The Media Sharing Subsystem

To prevent large file transfers from clogging the real-time messaging pipeline and introducing latency, media sharing is handled by a completely decoupled subsystem.12 This subsystem is managed by a separate fleet of servers, often referred to as MMS (multimedia) servers.12 This design embodies the "pointer economy"—a core architectural pattern where the main chat servers handle only lightweight pointers (like URLs or hashes) rather than the heavy data itself.

The upload and download process is as follows:

1. **Client-Side Preparation:** The sender's client first compresses and encrypts the media file (image, video, document) locally on the device.14
    
2. **Direct Upload to Blob Storage:** The client then makes a request to the WhatsApp backend to get a temporary, authorized upload URL. It uses this pre-signed URL to upload the encrypted media blob directly to a dedicated object storage service (a system analogous to AWS S3 or Google Cloud Storage) via an HTTP POST request.14 This critical step offloads the entire bandwidth-intensive file transfer from the core chat servers.
    
3. **Pointer Generation:** The object storage service, upon successful upload, returns a unique identifier or URL for the stored blob.
    
4. **Pointer Transmission:** The sender's client then composes a standard, lightweight chat message that contains this media pointer. This message is sent through the normal, highly-optimized real-time messaging pipeline.14
    
5. **Recipient Download:** The recipient's client receives this pointer message. It then uses the contained URL to download the encrypted media blob directly from the object storage service, again bypassing the core chat servers.14 After downloading, the client decrypts the file locally for the user to view.
    

The HTTP traffic for these media transfers is handled by YAWS (Yet Another Web Server), a high-performance web server written in Erlang. This choice maintains the technological homogeneity of the backend stack, allowing the same underlying performance optimizations and operational tooling to be applied to both the real-time and HTTP-based components of the system.2

## The Fortress: Security and Encryption Architecture

Security, specifically end-to-end encryption (E2EE), is not merely a feature of WhatsApp but the central, defining constraint around which the entire system is architected. This commitment to content privacy fundamentally dictates what the server can and cannot do, forcing intelligence and cryptographic operations to the client-side "edge."

### The Signal Protocol: The Gold Standard for E2EE

In a landmark move for user privacy, WhatsApp completed the integration of the open-source Signal Protocol, developed by Open Whisper Systems, in April 2016.43 This provided default E2EE for every form of communication on the platform—including one-to-one chats, group chats, voice and video calls, and file sharing.24

The core property of E2EE is that it ensures only the communicating users can read the content of their messages. The encryption and decryption operations happen exclusively on the users' devices.46 The keys required to decrypt the messages are held only by the participating clients and are never transmitted to or stored on WhatsApp's servers. As a result, not even WhatsApp or its parent company, Meta, can access the content of the communications, rendering them effectively blind to the data that transits their network.2

### Key Exchange, The Double Ratchet, and Forward Secrecy

The Signal Protocol's strength comes from a combination of advanced cryptographic mechanisms.

- **Asynchronous Key Exchange (X3DH):** When two users communicate for the first time, they do not need to be online simultaneously to establish a secure session. The protocol uses an asynchronous key exchange mechanism called the Extended Triple Diffie-Hellman (X3DH) handshake. Each client generates and uploads a set of signed "pre-keys" to the server. To initiate a session, the sender fetches the recipient's "pre-key bundle" from the server and uses it to derive a shared secret key, even if the recipient is offline.38
    
- **The Double Ratchet Algorithm:** Once a secure session is established, the protocol does not use the same key to encrypt every message. Instead, it employs the Double Ratchet algorithm to generate a new, ephemeral encryption key for each and every message sent. This provides remarkable security properties. The "ratchet" has two components:
    
    1. **Symmetric-key Ratchet:** For each message sent, a new message key is derived from the previous one using a one-way Key Derivation Function (KDF), typically based on HMAC-SHA256. Because the function is one-way, knowing the key for message N does not allow an attacker to compute the key for message N-1. This provides **forward secrecy** within a session.48
        
    2. **Diffie-Hellman Ratchet:** Periodically, as messages are exchanged, the participants also perform a new Diffie-Hellman (DH) key exchange and mix the result into the session state. This provides a property known as **"self-healing"** or post-compromise security. If an attacker manages to compromise a user's session keys at a specific point in time, the next DH ratchet turn will establish a new secret that the attacker cannot derive, effectively "healing" the session from the compromise and protecting all future messages.38
        
- **Forward Secrecy:** This is one of the most critical security guarantees of the protocol. It ensures that the compromise of a user's long-term identity keys does not compromise the confidentiality of their past conversations. Because past messages were encrypted with short-lived, ephemeral keys that have since been discarded, an attacker who steals a user's phone today cannot use its keys to decrypt a backup of network traffic they may have recorded weeks or months ago.36
    

### The Custom Binary Protocol

The communication between the WhatsApp client and server is not conducted in plaintext or a human-readable format like JSON. For maximum efficiency over mobile networks, it uses a custom binary protocol.20 Reverse-engineering efforts have revealed that messages are first serialized into a compact binary format, conceptually similar to Google's Protocol Buffers, and then wrapped in an additional layer of transport encryption before being sent over the WebSocket or TCP connection.19

A typical encrypted message frame transmitted from the client has a distinct structure: `message_tag,<signature><iv><ciphertext>`.20

- `message_tag`: A client-generated tag, often a timestamp or sequence number, used to correlate requests and responses.
    
- `<signature>`: A 32-byte HMAC-SHA256 signature of the encrypted payload. This ensures the integrity and authenticity of the message, verifying that it has not been tampered with in transit between the client and server.
    
- `<iv>`: A 16-byte initialization vector used for the AES-CBC encryption algorithm.
    
- `<ciphertext>`: The end-to-end encrypted payload, which itself contains the Signal Protocol-encrypted message.
    

To further reduce the data footprint, the binary protocol makes heavy use of tokenization. Common strings used in the protocol's commands and attributes (e.g., "message", "user", "query") are replaced with single-byte integer tokens, minimizing the overhead of each transmission.20

The decision to implement E2EE is the single most defining constraint on WhatsApp's system architecture. It dictates that the server must be designed as a "dumb" but highly efficient transport layer for opaque, encrypted blobs of data. This forces complex functionality to the client "edge." The client is responsible for all cryptographic operations, local data storage and indexing (in SQLite), and implementing features like local search. This constraint directly shapes the media sharing and group chat architectures, which are designed to work without the server ever having access to plaintext content.

This design also highlights a fundamental nuance in the concept of "privacy." While WhatsApp provides world-class _content privacy_ through its E2EE implementation, like any centralized communication service, it necessarily involves a trade-off in _metadata privacy_. The server must know who is talking to whom, when they are talking, and from what IP addresses in order to route messages. It manages group membership lists and presence status.29 While protocols like Signal's "Sealed Sender" aim to minimize even this metadata exposure, the fundamental architecture of a centralized service requires the server to process this routing information. This represents an inherent tension between providing a seamless, large-scale service and achieving absolute metadata privacy.

## A New Paradigm: The Architectural Shift to Multi-Device Support

For years, WhatsApp's architecture was anchored to a single, fundamental principle: the user's phone was the one and only source of truth. The launch of a true multi-device capability in 2021 represented the most significant re-architecture in the service's recent history, requiring a complete rethinking of identity, security, and data synchronization.

### The Old Model: The Phone as the Source of Truth

In the original architecture, companion devices like WhatsApp Web or the desktop clients were not independent. They functioned as "mirrors" or thin clients, securely proxying all communications through the user's primary phone.40 A companion device would establish a secure connection to the phone, which would then relay messages to and from the WhatsApp servers.

This model, while simple to implement, had significant usability drawbacks. If the user's phone ran out of battery, lost its internet connection, or if the operating system aggressively killed the WhatsApp application process, all companion devices would immediately cease to function.40 Furthermore, the architecture only supported the use of a single companion device at a time.40

### The New Model: Independent Device Identity

The modern multi-device architecture dismantles the phone-centric model. The core conceptual change is the shift from `1 user = 1 identity` to `1 user = N identities`. Each of a user's devices—their primary phone plus up to four non-phone companion devices—is now a first-class citizen with its own unique cryptographic identity key.40

The role of the server has evolved to support this new identity model. The WhatsApp server now maintains a mapping between a single user account and the list of all public identity keys for their registered devices. When a sender wants to transmit a message, their client must first query the server to get the list of all registered devices for each recipient. This device list is essential for the new synchronization model.40

### The Client-Fanout Model for Message Synchronization

To ensure all of a user's devices remain synchronized without a central phone acting as a hub, WhatsApp adopted a **client-fanout** approach for message delivery.40 This model pushes the responsibility for message duplication from the server to the sending client.

When a user sends a message from any of their registered devices (e.g., their laptop), the client application on that device performs the following steps:

1. It establishes individual, pairwise E2EE sessions with _every_ device of the recipient(s).
    
2. Crucially, it also establishes pairwise E2EE sessions with all of the sender's _own_ other linked devices.
    
3. It then encrypts the message payload separately for each of these target devices, creating N unique ciphertexts.
    
4. Finally, it transmits all N encrypted messages to the WhatsApp server. The server acts as a simple routing layer, delivering each encrypted message to its intended destination device.
    

This architecture ensures that every device receives its own copy of a message directly and can operate fully independently, even if the user's phone is turned off.40

### Maintaining E2EE Integrity Across Devices

This new architecture required novel solutions to maintain the integrity of end-to-end encryption across a multi-device environment.

- **Secure Device Linking:** The process of linking a new companion device is a critical security ceremony. It is typically initiated by scanning a QR code displayed on the companion device with the user's primary phone. This action, which can be protected by on-device biometric authentication, securely transfers the necessary identity keys to establish the new device as a trusted part of the user's account.40
    
- **Chat History Synchronization:** To provide a seamless user experience, recent chat history must be available on a newly linked device. This is achieved through a one-time transfer. The primary phone encrypts a bundle of recent messages and transfers this large, encrypted blob to the new device. The decryption key for this blob is sent separately via a standard E2EE message. Once the new device has successfully decrypted and stored this history in its own local database, the temporary keys are deleted.40 From that point on, it builds its history independently.
    
- **Multi-Device Group Calls:** Group calls also had to be re-architected. In the multi-device world, when a group call is initiated, the server randomly selects one _device_ from all active participants to act as the key generator. This device generates the call's SRTP master secret key and then securely distributes it to all other participating devices using their established pairwise E2EE sessions. This key is re-generated whenever a participant joins or leaves the call.40
    

The shift to a multi-device architecture illustrates how a single, user-facing feature requirement—"use WhatsApp when your phone is off"—can trigger a cascade of fundamental architectural changes. This "identity explosion" from one key per user to many keys per user forced a complete re-platforming of the service's core concepts of identity, security, and message routing. It also demonstrates a significant trend in modern systems: pushing complexity and workload to the client. The new architecture grants users more convenience and a more robust E2EE model, but it does so at the cost of offloading the significant computational and network burden of the fan-out model from the server to the client application.

|Architectural Aspect|Legacy Architecture (Phone-Centric)|Modern Architecture (Multi-Device)|
|---|---|---|
|**Source of Truth**|The user's primary phone is the single source of truth.40|Each device is an independent client; the server syncs state across them.40|
|**Device Identity**|A single cryptographic identity key is tied to the user's account/phone.40|Each device (phone + companions) has its own unique identity key.40|
|**Message Synchronization**|Phone-as-proxy: The phone relays all messages to/from companion devices.40|Client-Fanout: The sending client encrypts and sends a copy of the message to every recipient device and its own other devices.40|
|**Offline Functionality**|Companion devices are non-functional if the primary phone is offline.40|Companion devices can send and receive messages independently, even if the phone is offline.40|
|**Group Chat Fan-Out**|Primarily Server-Side Fan-Out, where the server distributes a single ciphertext to group members.36|Client-Side Fan-Out, where the client encrypts and sends the message to every device in the group.40|

## The Art of Performance: WhatsApp's Maniacal Focus on Optimization

WhatsApp's ability to serve billions of users with a famously small engineering team is not just a result of a well-chosen tech stack, but also of a relentless and unusually deep culture of performance optimization. Their approach goes far beyond typical application-level tuning, extending down into the virtual machine and the operating system kernel itself.

### The Philosophy of Vertical Scaling

The core of WhatsApp's scaling strategy has been to scale _up_ before scaling _out_. They have consistently prioritized maximizing the performance of individual, powerful servers—a practice known as vertical scaling—rather than distributing load across a vast fleet of smaller, commodity machines.7

The primary motivation for this philosophy is the reduction of operational complexity. The engineering team operates on the principle that "operational complexity scales with the number of nodes, not the number of cores".12 Managing, monitoring, patching, and deploying to a fleet of 500 highly-tuned servers is considered significantly more manageable than doing so for 5,000 generic ones. This forces a focus on efficiency and getting the absolute maximum performance out of every single machine. This choice has a symbiotic relationship with their technology stack; the decision to scale vertically was only feasible because they chose an open, "hackable" stack (Erlang/OTP and FreeBSD) that they could modify at a fundamental level to meet their extreme performance demands.10

### Engineering at the Source: Key Patches to BEAM, FreeBSD, and Mnesia

Unlike most organizations that treat their language runtime and operating system as immutable black boxes, the WhatsApp engineering team viewed them as integral parts of their own system, subject to the same scrutiny and optimization as their application code. They identified and patched numerous bottlenecks at the lowest levels of the stack.11

#### FreeBSD Kernel Optimizations

To handle millions of TCP connections per host, the team made several critical changes to the FreeBSD kernel 11:

- **Network Performance:** They backported the `igb` network driver from a newer version of FreeBSD to fix stability issues with multi-queue network interface cards (NICs) under heavy load. They also significantly increased the size of the kernel's hash table for Protocol Control Blocks (PCBs) to accelerate the lookup of network connection information.
    
- **Timers:** They backported a more efficient, lower-overhead hardware timer counter (TSE) to reduce the CPU cost of getting the system time, an operation performed constantly in a messaging system.
    

#### BEAM Virtual Machine Optimizations

The journey from supporting 200,000 connections per server to over 2 million was almost entirely driven by fixing contention issues within the BEAM VM itself.13 Key patches included 11:

- **Lock Contention:** A major bottleneck was a single global lock on the `timeofday` function, which was hit every time a message was delivered. They eliminated this by optimizing the VM's internal timer wheel data structures.
    
- **Memory Management:** They patched the BEAM's memory allocators (`mseg`) to be per-scheduler, avoiding global contention. They also tuned allocation strategies to encourage the OS to use "super pages," which reduces the rate of Translation Lookaside Buffer (TLB) thrashing and improves memory throughput. To handle load spikes, they implemented garbage collection (GC) throttling, which would pause the GC for processes with very large message mailboxes until the queue size shrank, preventing GC from destabilizing the entire node.
    
- **Scheduler Tuning:** They ran the BEAM process at a real-time scheduling priority to ensure that less critical system tasks like cron jobs could not preempt the core message-processing work. They also finely tuned scheduler parameters like the wake-up threshold (`+swt low`) and spin counts to ensure schedulers would wake promptly when work was available but not waste CPU cycles spinning unnecessarily.
    

#### Mnesia and OTP Optimizations

The database and standard libraries were also subject to heavy modification 11:

- **Replication Throughput:** To overcome bottlenecks in Mnesia's replication, they patched the transaction manager (`mnesia_tm`) to dispatch `async_dirty` transactions to separate, per-table processes, allowing replication for different tables to proceed in parallel.
    
- **Disk I/O:** They added a patch that allowed Mnesia's on-disk storage directory to be split across multiple physical drives, increasing disk I/O throughput, which was especially critical during database loading and recovery.
    
- **Lookup Performance:** They identified and fixed a flaw in a hashing function that was causing extremely long hash chains and slow lookups in heavily fragmented Mnesia tables, resulting in a 4x performance improvement with a two-line code change.
    

### Application-Level Performance Patterns

In addition to platform-level tuning, the application architecture itself is designed for performance.

- **Asynchronicity and Decoupling:** The system makes extensive use of asynchronous message passing to ensure that slow operations, particularly I/O, do not block the main processing path. Services are decoupled and communicate via messages, using custom patterns like `gen_industry`—a multi-dispatcher version of the standard `gen_server`—to parallelize work and prevent head-of-line blocking.12
    
- **Intelligent Caching:** For the offline message store, they implemented a write-back cache. This allowed messages destined for online users to be delivered immediately from memory, even if the underlying disk system was backlogged due to a load spike. This cache achieved a 98% hit rate and provided a crucial buffer to smooth out I/O performance.12 The team also recognized that a small number of "heavy users"—those in a large number of very active groups—could pollute the cache. They implemented logic to proactively identify and evict these users' data from the cache to preserve its effectiveness for the general user population.12
    

This approach redefines the scope of "system design." For the WhatsApp team, the system is not just the application code, but the entire stack from the hardware drivers up. Where a conventional team might address a bottleneck by adding more servers, the WhatsApp team's first instinct was to attach a debugger to the VM, identify the specific lock contention, and patch the C source code of the runtime itself. This holistic, full-stack ownership is exceptionally rare and stands as a key differentiator in their remarkable engineering achievement.

## Conclusion: Enduring Lessons from WhatsApp's Engineering Journey

The architecture of WhatsApp is a compelling narrative of engineering pragmatism, deep technical expertise, and a relentless focus on a core mission. Its success provides several enduring lessons for building systems at a global scale.

First is the profound impact of a purpose-built, homogenous technology stack. The choice of Erlang/OTP was not arbitrary; it was a deliberate selection of a toolset designed for the precise challenges of massive concurrency and extreme fault tolerance. This choice created a virtuous cycle: the use of Erlang led to the adoption of Mnesia, `ejabberd`, and YAWS, creating a cohesive ecosystem that amplified developer productivity and allowed a small team to achieve what would normally require a much larger one.

Second is the strategic value of vertical scaling as a means of managing operational complexity. In an industry often dominated by the mantra of horizontal scaling with commodity hardware, WhatsApp's focus on maximizing the performance of fewer, more powerful nodes demonstrates a different path. This philosophy, encapsulated in the idea that complexity scales with the number of nodes rather than cores, forced a culture of deep optimization but yielded a system that was more manageable and operationally lean.

Third, the implementation of end-to-end encryption serves as a powerful case study in how a non-functional requirement can become the single most defining architectural constraint. The decision to make the server blind to user content fundamentally shaped the entire system, pushing intelligence to the client-side edge and dictating the design of every major feature, from media sharing to group chats. It underscores the principle that security is not a layer to be added on, but a foundational property that must be designed in from the beginning.

Finally, the architectural pivot to support a true multi-device experience illustrates that no system design, no matter how successful, is ever truly finished. It shows that evolving user expectations can and will force fundamental re-architecting of even the most scaled and stable systems. The shift from a phone-centric model to one of independent device identities was a monumental undertaking that touched every part of the platform, from identity management to the core messaging protocol. It is a testament to the team's ability to adapt and evolve, proving that the capacity for radical change is as important to long-term success as the initial brilliance of the design.

In synthesis, WhatsApp's backend is more than just a collection of technologies; it is the physical manifestation of an engineering culture that values simplicity, full-stack ownership, and an unwavering focus on reliability at scale.

[

![](https://t2.gstatic.com/faviconV2?url=https://en.wikipedia.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

en.wikipedia.org

Erlang (programming language) - Wikipedia

Opens in a new window](https://en.wikipedia.org/wiki/Erlang_\(programming_language\))[

![](https://t0.gstatic.com/faviconV2?url=https://www.erlang-factory.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

erlang-factory.com

efsf2012-whatsapp-scaling.pdf - Erlang Factory

Opens in a new window](https://www.erlang-factory.com/upload/presentations/558/efsf2012-whatsapp-scaling.pdf)[

![](https://t1.gstatic.com/faviconV2?url=https://blog.stenmans.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

blog.stenmans.org

The BEAM Book: Understanding the Erlang Runtime System - Happi

Opens in a new window](https://blog.stenmans.org/theBeamBook/)[

![](https://t2.gstatic.com/faviconV2?url=https://blog.quastor.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

blog.quastor.org

How WhatsApp scaled to 1 billion users with only 50 engineers - Quastor

Opens in a new window](https://blog.quastor.org/p/whatsapp-scaled-1-billion-users-50-engineers)[

![](https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

reddit.com

Designed WhatsApp's Chat System on Paper—Here's What Blew My Mind - Reddit

Opens in a new window](https://www.reddit.com/r/softwarearchitecture/comments/1jz98b6/designed_whatsapps_chat_system_on_paperheres_what/)[

![](https://t0.gstatic.com/faviconV2?url=https://stackoverflow.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

stackoverflow.com

What is the technology behind wechat, whatsapp and other messenger apps? [closed]

Opens in a new window](https://stackoverflow.com/questions/19640703/what-is-the-technology-behind-wechat-whatsapp-and-other-messenger-apps)[

![](https://t0.gstatic.com/faviconV2?url=https://dev.to/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

dev.to

WhatsApp System Design - DEV Community

Opens in a new window](https://dev.to/sborhade/whatsapp-system-design-1g5o)[

![](https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

reddit.com

How WhatsApp scaled to 1 billion users with only 50 engineers : r/programming - Reddit

Opens in a new window](https://www.reddit.com/r/programming/comments/qfbs22/how_whatsapp_scaled_to_1_billion_users_with_only/)[

![](https://t0.gstatic.com/faviconV2?url=https://kindsonthegenius.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

kindsonthegenius.com

WhatsApp System Design (Interview Question) - Kindson The Genius

Opens in a new window](https://kindsonthegenius.com/system-design/system-design-whatsapp-system-design-interview-question/)[

![](https://t2.gstatic.com/faviconV2?url=https://forums.freebsd.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

forums.freebsd.org

WhatsApp on FreeBSD

Opens in a new window](https://forums.freebsd.org/threads/whatsapp-on-freebsd.87908/)[

![](https://t1.gstatic.com/faviconV2?url=https://highscalability.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

highscalability.com

The WhatsApp Architecture Facebook Bought For $19 Billion - High ...

Opens in a new window](https://highscalability.com/the-whatsapp-architecture-facebook-bought-for-19-billion/)[

![](https://t1.gstatic.com/faviconV2?url=https://highscalability.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

highscalability.com

How WhatsApp Grew to Nearly 500 Million Users, 11,000 cores, and ...

Opens in a new window](https://highscalability.com/how-whatsapp-grew-to-nearly-500-million-users-11000-cores-an/)[

![](https://t1.gstatic.com/faviconV2?url=https://www.geeksforgeeks.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

geeksforgeeks.org

How WhatsApp handles 50 billion messages a day? - GeeksforGeeks

Opens in a new window](https://www.geeksforgeeks.org/system-design/how-whatsapp-handles-50-billion-messages-a-day/)[

![](https://t3.gstatic.com/faviconV2?url=https://faq.whatsapp.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

faq.whatsapp.com

About end-to-end encryption | WhatsApp Help Center

Opens in a new window](https://faq.whatsapp.com/general/security-and-privacy/end-to-end-encryption)[

![](https://t1.gstatic.com/faviconV2?url=https://github.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

github.com

lharries/whatsapp-mcp: WhatsApp MCP server - GitHub

Opens in a new window](https://github.com/lharries/whatsapp-mcp)[

![](https://t3.gstatic.com/faviconV2?url=https://blog.cryptographyengineering.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

blog.cryptographyengineering.com

Attack of the Week: Group Messaging in WhatsApp and Signal - A Few Thoughts on Cryptographic Engineering

Opens in a new window](https://blog.cryptographyengineering.com/2018/01/10/attack-of-the-week-group-messaging-in-whatsapp-and-signal/)[

![](https://t0.gstatic.com/faviconV2?url=https://dev.to/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

dev.to

Engineering Connectivity: Navigating the System Design of WhatsApp - DEV Community

Opens in a new window](https://dev.to/binoy123/engineering-connectivity-navigating-the-system-design-of-whatsapp-49o3)[

![](https://t1.gstatic.com/faviconV2?url=https://news.ycombinator.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

news.ycombinator.com

Here's how WhatsApp group messaging works: membership is maintained by the serve... | Hacker News

Opens in a new window](https://news.ycombinator.com/item?id=16117487)[

![](https://t3.gstatic.com/faviconV2?url=https://www.karanpratapsingh.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

karanpratapsingh.com

WhatsApp | System Design - Karan Pratap Singh

Opens in a new window](https://www.karanpratapsingh.com/courses/system-design/whatsapp)[

![](https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

reddit.com

8 Reasons Why WhatsApp Was Able to Support 50 Billion Messages a Day With Only 32 Engineers : r/programming - Reddit

Opens in a new window](https://www.reddit.com/r/programming/comments/162jwxo/8_reasons_why_whatsapp_was_able_to_support_50/)[

![](https://t1.gstatic.com/faviconV2?url=https://github.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

github.com

whatsapp-decoding/PROTOCOL.md at master - GitHub

Opens in a new window](https://github.com/Enrico204/whatsapp-decoding/blob/master/PROTOCOL.md)[

![](https://t2.gstatic.com/faviconV2?url=https://en.wikipedia.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

en.wikipedia.org

Signal Protocol - Wikipedia

Opens in a new window](https://en.wikipedia.org/wiki/Signal_Protocol)[

![](https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

reddit.com

What's the main advantage of the Signal Protocol? - Reddit

Opens in a new window](https://www.reddit.com/r/signal/comments/1bo0z2r/whats_the_main_advantage_of_the_signal_protocol/)[

![](https://t1.gstatic.com/faviconV2?url=https://github.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

github.com

christophhagen/LibSignalProtocolSwift: A Swift implementation of the Signal Protocol

Opens in a new window](https://github.com/christophhagen/LibSignalProtocolSwift)[

![](https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

reddit.com

What's stopping Whatsapp from storing the private key in their servers? : r/crypto - Reddit

Opens in a new window](https://www.reddit.com/r/crypto/comments/l8pdgo/whats_stopping_whatsapp_from_storing_the_private/)[

![](https://t1.gstatic.com/faviconV2?url=https://highscalability.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

highscalability.com

Designing WhatsApp - High Scalability -

Opens in a new window](https://highscalability.com/designing-whatsapp/)[

![](https://t2.gstatic.com/faviconV2?url=https://www.cometchat.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

cometchat.com

Understanding WhatsApp's Architecture & System Design - CometChat

Opens in a new window](https://www.cometchat.com/blog/whatsapps-architecture-and-system-design)[

![](https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

reddit.com

Help me explain how whatsapp's file sharing works and how can I achieve it using Laravel

Opens in a new window](https://www.reddit.com/r/laravel/comments/hgul06/help_me_explain_how_whatsapps_file_sharing_works/)[

![](https://t1.gstatic.com/faviconV2?url=https://intuji.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

intuji.com

WhatsApp Tech Stack Explored — The Tech Behind Series - Intuji

Opens in a new window](https://intuji.com/whatsapp-tech-stack-explored/)[

![](https://t0.gstatic.com/faviconV2?url=https://mycodeplex.wordpress.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

mycodeplex.wordpress.com

Inside Of WhatsApp : Part 1 - Sunny Patel - WordPress.com

Opens in a new window](https://mycodeplex.wordpress.com/2016/03/08/inside-of-whatsapp-part-1/)[

![](https://t3.gstatic.com/faviconV2?url=https://www.contus.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

contus.com

How to Build a Chat App like WhatsApp, Telegram & Slack? - CONTUS Tech

Opens in a new window](https://www.contus.com/blog/how-whatsapp-works-technically-and-how-to-build-an-app-similar-to-it/)[

![](https://t0.gstatic.com/faviconV2?url=https://www.rst.software/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

rst.software

22 companies using XMPP and ejabberd to build instant messaging services | RST Software

Opens in a new window](https://www.rst.software/blog/22-companies-using-xmpp-and-ejabberd-to-build-instant-messaging-services)[

![](https://t0.gstatic.com/faviconV2?url=https://courses.csail.mit.edu/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

courses.csail.mit.edu

WhatsApp Security Paper Analysis - courses

Opens in a new window](https://courses.csail.mit.edu/6.857/2016/files/36.pdf)[

![](https://t1.gstatic.com/faviconV2?url=https://www.geeksforgeeks.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

geeksforgeeks.org

Designing Whatsapp Messenger | System Design - GeeksforGeeks

Opens in a new window](https://www.geeksforgeeks.org/system-design/designing-whatsapp-messenger-system-design/)[

![](https://t1.gstatic.com/faviconV2?url=https://i.blackhat.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

i.blackhat.com

REVERSE ENGINEERING WHATSAPP ENCRYPTION FOR CHAT MANIPULATION AND MORE - Black Hat

Opens in a new window](https://i.blackhat.com/USA-19/Wednesday/us-19-Zaikin-Reverse-Engineering-WhatsApp-Encryption-For-Chat-Manipulation-And-More.pdf)[

![](https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

reddit.com

How Does the Backend of Apps Like WhatsApp Work? : r/developersIndia - Reddit

Opens in a new window](https://www.reddit.com/r/developersIndia/comments/1g52yjy/how_does_the_backend_of_apps_like_whatsapp_work/)[

![](https://t1.gstatic.com/faviconV2?url=https://news.ycombinator.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

news.ycombinator.com

WhatsApp was built almost entirely in Erlang + Mnesia. I honestly don't think - Hacker News

Opens in a new window](https://news.ycombinator.com/item?id=15173885)[

![](https://t1.gstatic.com/faviconV2?url=https://elixirforum.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

elixirforum.com

Yaws vs. Cowboy (and Phoenix) - Chat / Discussions - Elixir Programming Language Forum

Opens in a new window](https://elixirforum.com/t/yaws-vs-cowboy-and-phoenix/3348)[

![](https://t1.gstatic.com/faviconV2?url=https://elixirforum.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

elixirforum.com

Mnesia vs Cassandra (vs CouchDB vs ...) - your thoughts?

Opens in a new window](https://elixirforum.com/t/mnesia-vs-cassandra-vs-couchdb-vs-your-thoughts/14593)[

![](https://t3.gstatic.com/faviconV2?url=https://www.infoq.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

infoq.com

WhatsApp Adopts the Signal Protocol for Secure Multi-Device Communication - InfoQ

Opens in a new window](https://www.infoq.com/news/2021/07/WhatsApp-signal-protocol/)[

![](https://t1.gstatic.com/faviconV2?url=https://signal.org/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

signal.org

WhatsApp's Signal Protocol integration is now complete

Opens in a new window](https://signal.org/blog/whatsapp-complete/)[

![](https://t3.gstatic.com/faviconV2?url=https://www.hellointerview.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

hellointerview.com

Design a Messaging App Like WhatsApp - Hello Interview

Opens in a new window](https://www.hellointerview.com/learn/system-design/problem-breakdowns/whatsapp)[

![](https://t1.gstatic.com/faviconV2?url=https://github.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

github.com

cabol/erlbus: Simple, Distributed and Scalable PubSub Message Bus written in Erlang - GitHub

Opens in a new window](https://github.com/cabol/erlbus)[

![](https://t3.gstatic.com/faviconV2?url=https://talent500.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

talent500.com

How WhatsApp Powers 40 Billion Messages Daily - Talent500

Opens in a new window](https://talent500.com/blog/whatsapp-scalable-messaging-architecture/)[

![](https://t1.gstatic.com/faviconV2?url=https://pwskills.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

pwskills.com

Understanding System Design Whatsapp & Architecture - PW Skills

Opens in a new window](https://pwskills.com/blog/system-design-whatsapp/)[

![](https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

reddit.com

Whatsapp system design : r/softwarearchitecture - Reddit

Opens in a new window](https://www.reddit.com/r/softwarearchitecture/comments/109hyaq/whatsapp_system_design/)[

![](https://t0.gstatic.com/faviconV2?url=https://www.youtube.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

youtube.com

System Design Interview: Design Whatsapp w/ a Ex-Meta Senior Manager - YouTube

Opens in a new window](https://www.youtube.com/watch?v=cr6p0n0N-VA&pp=0gcJCf0Ao7VqN5tD)[

![](https://t3.gstatic.com/faviconV2?url=https://engineering.fb.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

engineering.fb.com

How WhatsApp enables multi-device capability - Engineering at Meta

Opens in a new window](https://engineering.fb.com/2021/07/14/security/whatsapp-multi-device/)[

![](https://t2.gstatic.com/faviconV2?url=https://security.stackexchange.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

security.stackexchange.com

encryption - How does WhatsApp's new group chat protocol work ...

Opens in a new window](https://security.stackexchange.com/questions/119633/how-does-whatsapps-new-group-chat-protocol-work-and-what-security-properties-do)[

![](https://t3.gstatic.com/faviconV2?url=https://digitalcommons.newhaven.edu/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

digitalcommons.newhaven.edu

WhatsApp Network Forensics: Decrypting and Understanding the ...

Opens in a new window](https://digitalcommons.newhaven.edu/cgi/viewcontent.cgi?article=1049&context=electricalcomputerengineering-facpubs)[

![](https://t1.gstatic.com/faviconV2?url=https://github.com/&client=BARD&type=FAVICON&size=256&fallback_opts=TYPE,SIZE,URL)

github.com

sigalor/whatsapp-web-reveng: Reverse engineering ... - GitHub





](https://github.com/sigalor/whatsapp-web-reveng)