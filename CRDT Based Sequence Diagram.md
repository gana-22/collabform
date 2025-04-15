sequenceDiagram
    participant Client
    participant Load Balancer
    participant Server Cluster
    participant MongoDB Cluster
    note over MongoDB Cluster: Sharded Cluster
    participant Kafka
    participant Kafka Consumer
    participant PostgreSQL
    participant Redis

    %% Authentication & Authorization
    Client->>Load Balancer: Authentication Request (Username/Password)
    Load Balancer->>Server Cluster: Distribute Authentication Request
    Server Cluster->>PostgreSQL: Verify Credentials
    PostgreSQL-->>Server Cluster: Authentication Result
    Server Cluster-->>Load Balancer: Authentication Result
    Load Balancer-->>Client: JWT Token
    Client->>Load Balancer: Authorized Request (with JWT)
    Load Balancer->>Server Cluster: Distribute Authorized Request
    Server Cluster->>PostgreSQL: Verify Authorization

    %% Initial Document Load
    Client->>Load Balancer: Load Document Request (Document ID, JWT)
    Client->>Load Balancer: WebSocket Connection
    note over Load Balancer: Sticky Sessions Enabled
    Load Balancer->>Server Cluster: Distribute Document Request
    Load Balancer->>Server Cluster: Distribute WebSocket Connection
    Server Cluster->>PostgreSQL: Verify Authorization
    Server Cluster->>MongoDB Cluster: Retrieve Document (Document ID)
    MongoDB Cluster-->>Server Cluster: Serialized Document Data
    Server Cluster->>Client: Deserialized Document Data

    %% CRDT Operation Processing
    Client->>Client: Update Rendered Document
    Client->>Client: Apply Changes Onto In-Memory Document
    Client->>Client: Summarise CRDT Operations & Event Parse
    Client->>Server Cluster: Broadcast CRDT Operation (with Version Vector, Unique ID & Document ID)
    Server Cluster->>Server Cluster: Idempotency Check & CRDT Merge on In-Memory Document
    Server Cluster->>MongoDB Cluster: Obtain Distributed Lock
    MongoDB Cluster-->>Server Cluster: Lock Acquired
    Server Cluster->>MongoDB Cluster: Database Write (CRDT Raw Operation)
    Server Cluster->>MongoDB Cluster: Store CRDT Data
    Server Cluster->>Kafka: Publish CRDT Operation (with Version Vector, Unique ID & Document ID)
    Server Cluster->>MongoDB Cluster: Release Distributed Lock
    MongoDB Cluster-->>Server Cluster: Lock Released
    
    %% kafka Consumer Processing
    Kafka->>Kafka Consumer: CRDT Operation (with Version Vector, Unique ID & Document ID)
    Kafka Consumer->>Kafka Consumer: Idempotency Check & CRDT Merge
    Kafka Consumer->>PostgreSQL: Store Version Vector, Unique ID & Document ID
    Kafka Consumer->>MongoDB Cluster: Periodically Create Snapshot
    Kafka Consumer->>Redis: Cache Snapshot

    %% Client Processing
    Server Cluster->>Client: CRDT Operation (with Version Vector, Unique ID & Document ID)
    Client->>Client: Idempotency Check & CRDT Merge on In-Memory Document
    Client->>Client: Document Rendering from In-Memory

    %% Request Document Version/History
    opt Request Document Version
        Client->>Load Balancer: Request Document Version
        Load Balancer->>Server Cluster: Request Document Version
        Server Cluster->>Redis: Check for Snapshot
    end
    alt Snapshot in Redis
        Server Cluster->>Load Balancer: Snapshot from Redis
        Load Balancer->>Client: Snapshot from Redis
    else Snapshot not in Redis
        Server Cluster->>MongoDB Cluster: Retrieve Snapshot
        Server Cluster->>Redis: Cache Snapshot
        Server Cluster->>Load Balancer: Snapshot from MongoDB
        Load Balancer->>Client: Snapshot from MongoDB
    end

    %% Kafka Failure Recovery
    opt Kafka Failure
        Server Cluster->>MongoDB Cluster: Retrieve Pending CRDT Operations
        Server Cluster->>Server Cluster: Apply Pending CRDT Operations (In-Memory)
        Server Cluster->>PostgreSQL: Store Version Vector, Unique ID & Document ID
    end

    %% Server Faiure Recovery
    opt Server Failure
        Server Cluster->>PostgreSQL: Retrieve Last Processed Version Vector
        Server Cluster->>MongoDB Cluster: Retrieve Pending CRDT Operations
        Server Cluster->>MongoDB Cluster: Retrieve Document (Based on Document ID)
        MongoDB Cluster-->>Server Cluster: Serialized Document Data
        Server Cluster->>Server Cluster: Idempotency Check & CRDT Merge
        Server Cluster->>MongoDB Cluster: Obtain Distributed Lock
        MongoDB Cluster-->>Server Cluster: Lock Acquired
        Server Cluster->>MongoDB Cluster: Database Write (CRDT Raw Operation)
        Server Cluster->>MongoDB Cluster: Serialize Document Data & Store
        Server Cluster->>Kafka: Publish CRDT Operation (with Version Vector, Unique ID & Document ID)
        Server Cluster->>MongoDB Cluster: Release Distributed Lock
        MongoDB Cluster-->>Server Cluster: Lock Released
    end
