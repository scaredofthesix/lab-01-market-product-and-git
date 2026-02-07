## Product Choice

**Product name:** Telegram  
**Website:** <https://telegram.org>  
**Short description:** Telegram is a cloud-based instant messaging platform providing fast and secure text messaging, voice and video calls, media sharing, group chats, and broadcast channels with over a billion users worldwide.

## Main components

![Telegram Component Diagram](../docs/diagrams/out/telegram/component-diagram/Component%20Diagram.svg)

**Component diagram code:**
[Telegram Component Diagram Code](../docs/diagrams/src/telegram/component-diagram.puml)

### Selected components

1. **MTProto Gateway (DC Entry)**
   Acts as the main entry point for client connections, terminating MTProto protocol, handling session routing, authentication handshakes, and forwarding validated requests to internal services.

2. **Message Handling Service**  
   Responsible for processing incoming messages, managing chat state, routing messages to recipients, and ensuring message ordering and delivery guarantees.

3. **Auth & Session Service**  
   Manages authentication, authorization, session lifecycle, device binding, and cryptographic key management.

4. **Notification / Updates Service**  
   Generates updates for clients and integrates with push notification systems to deliver real-time alerts for offline devices.

5. **Media & File Service**  
   Handles upload, storage, chunking, encryption, and distribution of media content using a distributed file system.

6. **Secret Chat Relay**  
   Routes encrypted peer-to-peer traffic for secret chats, acting as a transport layer without access to message plaintext.

## Data flow

![Telegram Sequence Diagram](../docs/diagrams/out/telegram/sequence-diagram/Sequence%20Diagram.svg)

**Sequence diagram code:**  
[Telegram Sequence Diagram Code](../docs/diagrams/src/telegram/sequence-diagram.puml)

### Selected flow: Sending a message

This flow describes how a user sends a standard cloud chat message.

1. The **Client Application** establishes a secure MTProto session with the **MTProto Gateway**.
2. The gateway authenticates the session using the **Auth & Session Service**.
3. The validated request is forwarded via **internal RPC (TL-schema)** to the **Message Handling Service**.
4. The Message Handling Service:
   - stores the message in the **Sharded Chat Database**,  
   - publishes events to the **Event Bus**,  
   - updates in-memory state using **Redis cache**.
5. The **Notification / Updates Service** consumes events and pushes updates to online devices or sends push notifications via **FCM/APNs**.
6. Recipient clients receive the update and fetch the new message state.
7.

**Components involved:**

- Client Applications  
- MTProto Gateway  
- Auth & Session Service  
- Message Handling Service  
- Notification / Updates Service  
- Sharded Chat DB  
- Redis Cache  
- Event Bus  

**Data exchanged:**

- Encrypted MTProto payload  
- Authentication tokens and session keys  
- Message payload and metadata  
- Chat state updates  
- Notification events
-

## Deployment

![Telegram Deployment Diagram](../docs/diagrams/out/telegram/deployment-diagram/Deployment%20Diagram.svg)

**Deployment diagram code:**  
[Telegram Deployment Diagram Code](../docs/diagrams/src/telegram/deployment-diagram.puml)

### Deployment description

- Client applications run on end-user devices (mobile, desktop, web).
- MTProto gateways and core backend services are deployed in geographically distributed Telegram data centers.
- Internal communication occurs via high-performance RPC inside private networks.
- Databases and caches are deployed in sharded and replicated clusters.
- Media content is stored in distributed file systems and delivered via global CDN nodes.
- External integrations include SMS providers and push notification services.

## Assumptions

1. I assume that messages are distributed across multiple servers to handle a large number of users at the same time.
2. I assume that caching is used to speed up access to frequently requested data and reduce server load.
3. I assume that Telegram uses background message queues to process notifications and updates asynchronously.

## Open questions

1. How does Telegram synchronize messages between user devices in real time across different regions?
2. How are secret chats technically separated from regular cloud chats?
3. What mechanisms are used to prevent service outages during peak load?
