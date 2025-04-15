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

    %% OT Operation Processing
    Client->>Server Cluster: OT Operation (Sequence Number & Operation ID)
    Server Cluster->>MongoDB Cluster: Obtain Distributed Lock
    MongoDB Cluster-->>Server Cluster: Lock Acquired
    Server Cluster->>MongoDB Cluster: Database Write (OT Raw Operation)
    Server Cluster->>Server Cluster: Idempotency Check & Ordering & Apply OT Operation
    Server Cluster->>MongoDB Cluster: Serialize Document Data & Store
    Server Cluster->>Kafka: Publish OT Operation (Sequence Number & Operation ID)
    Server Cluster->>MongoDB Cluster: Release Distributed Lock
    MongoDB Cluster-->>Server Cluster: Lock Released
    
    %% kafka Consumer Processing
    Kafka->>Kafka Consumer: OT Operation (Sequence Number & Operation ID)
    Kafka Consumer->>Kafka Consumer: Idempotency Check & Apply OT Operation
    Kafka Consumer->>PostgreSQL: Store Sequence Number & Operation ID
    Kafka Consumer->>MongoDB Cluster: Periodically Create Snapshot
    Kafka Consumer->>Redis: Cache Snapshot

    %% Client Processing
    Server Cluster->>Client: OT Operation (Sequence Number & Operation ID)
    Client->>Client: Idempotency Check & Apply OT Operation
    Client->>Client: Document Rendering

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
        Server Cluster->>MongoDB Cluster: Retrieve Pending OT Operations
        Server Cluster->>Server Cluster: Apply Pending OT Operations (In-Memory)
        Server Cluster->>PostgreSQL: Store Sequence Number & Operation ID
    end

    %% Server Faiure Recovery
    opt Server Failure
        Server Cluster->>PostgreSQL: Retrieve Last Processed Sequence Number & Operation ID
        Server Cluster->>MongoDB Cluster: Retrieve Pending OT Operations
        Server Cluster->>MongoDB Cluster: Retrieve Document (Based on Document ID)
        MongoDB Cluster-->>Server Cluster: Serialized Document Data
        Server Cluster->>MongoDB Cluster: Obtain Distributed Lock
        MongoDB Cluster-->>Server Cluster: Lock Acquired
        Server Cluster->>Server Cluster: Idempotency Check & Ordering & Apply OT Operation
        Server Cluster->>MongoDB Cluster: Serialize Document Data & Store
        Server Cluster->>Kafka: Publish OT Operation (Sequence Number & Operation ID)
        Server Cluster->>MongoDB Cluster: Release Distributed Lock
        MongoDB Cluster-->>Server Cluster: Lock Released
    end