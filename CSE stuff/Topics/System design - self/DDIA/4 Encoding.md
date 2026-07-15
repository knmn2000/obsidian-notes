----

Evolution is the only constant. Code changes, data remains.

# Compatibility

- **Backward Compatibility**: Newer code can read data written by older code (Easy).
- **Forward Compatibility**: Older code can read data written by newer code (Harder - need to ignore unknown fields).

---

# Data Encoding Formats

### 1. Language-Specific Formats

(e.g., Java [[serialization]], Python pickle)

- **Problem**: Tied to a specific language. Security risks (arbitrary code execution during deserialization). Versioning is usually a nightmare.
- **Verdict**: Avoid for cross-service communication.

### 2. JSON, XML, and CSV

- **Pros**: Human readable, widely supported.
- **Cons**:
  - Ambiguity with numbers (JSON doesn't distinguish integers and floats).
  - No support for binary strings (require Base64 encoding which increases size by 33%).
  - Schema support is optional and often clunky.
- **Binary JSON**: (BSON, MessagePack) They exist but don't save _that_ much space because they still store all field names ("userName", "orderId") in every single record.

---

# Binary Encoding (The Big Three)

To really save space and get performance, we need to stop storing field names.

## 1. Protocol Buffers (Protobuf) & Thrift

- They use a **Schema** to define data.
- Instead of field names, they use **Field Tags** (numbers like 1, 2, 3).
- **Space efficiency**: Very high. A field name "userName" (8 bytes) becomes a tag (1 byte).

### How they handle Evolution:

- **Forward Compatibility**: Older code sees a tag it doesn't recognize? It just skips it.
- **Backward Compatibility**: Newer code sees a missing tag? If it's marked as `optional`, no problem.
- **Rules**:
  - You can add/remove optional fields.
  - You can NEVER change a field tag.
  - If you add a required field, you break backward compatibility.

## 2. Apache Avro

- **The "No Tag" approach.**
- It doesn't store tags or field names. It just stores values in order.
- To read data, you **MUST** have the exact schema used to write it.

### How it handles Evolution:

- **Writer's Schema vs. Reader's Schema**: The reader and writer schemas don't have to be identical, they just need to be _compatible_.
- The library resolves differences by looking at field names.
- **Usecase**: Great for Hadoop/Big Data where you store millions of records in a file—you store the schema once at the start of the file.

---

# Modes of Data Flow

## 1. Data through Databases

- "Data is a message to your future self."
- A record might be written by code version 1 and read by code version 2 years later.
- **Problem**: If version 2 writes to a row, it might "clobber" (overwrite) new fields added by version 3 if not careful (Forward compatibility issue).

## 2. Data through Services ([[REST]] & [[RPC]])

- **[[REST]]**: Uses HTTP, URL-oriented, JSON/XML. Great for public APIs.
- **[[RPC]] (gRPC, Thrift)**: Tries to make a remote call look like a local function call.
  - **Problem**: Network is not like local memory. It can fail, timeout, or be slow.
  - **gRPC**: Uses Protobuf, better performance than REST.

## 3. Asynchronous Message Passing

(Kafka, RabbitMQ, NATS)

- **Pros**:
  - Decoupling (Sender doesn't need to know receiver).
  - Redelivery/Buffering (Receiver can be down).
  - One message to many consumers.
- **Evolution**: Same rules as DBs. You need to ensure consumers can handle older/newer message formats.

---

# Summary Trade-off

- **Textual (JSON)**: Good for "getting started" and public APIs. Expensive for high-throughput.
- **Binary (Protobuf/Avro)**: Best for internal microservices. Requires schema management but saves massive amounts of CPU and Bandwidth.
