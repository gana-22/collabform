flowchart TD
    Client[Document Editor Client]
    Respondent[Document Respondent]
    
    subgraph Gateway
        APIGW[API Gateway]
        LB[Load Balancer]
    end
    
    subgraph Services
        AuthService[Authentication Service]
        UserService[User Service]
        DocService[Document Service]
        DocVersionService[Document Versioning Service]
        CollabService[Real-time Collaboration Service]
        DistService[Distribution Service]
        RespService[Response Service]
        KafkaConsumer[Kafka Consumer Service]
    end
    
    subgraph Messaging
        Kafka[Kafka]
    end
    
    subgraph DataStores
        PG[(PostgreSQL)]
        Mongo[(MongoDB)]
        Redis[(Redis)]
    end
    
    %% Client Interactions
    Client -- "Authentication (REST)" --> APIGW
    Client -- "Document CRUD (REST)" --> APIGW
    Client -- "Real-time Collab (WebSocket)" --> APIGW
    Client -- "Form Distribution (REST)" --> APIGW
    Client -- "Document Versioning (REST)" --> APIGW
    
    Respondent -- "Access Form (REST)" --> APIGW
    Respondent -- "Submit Response (REST)" --> APIGW
    
    %% API Gateway Routing
    APIGW -- "Auth Requests" --> LB
    APIGW -- "User Requests" --> LB
    APIGW -- "Document Requests" --> LB
    APIGW -- "WebSocket Connection" --> LB
    APIGW -- "Distribution Requests" --> LB
    APIGW -- "Response Requests" --> LB
    APIGW -- "Version Requests" --> LB
    
    %% Load Balancer to Services
    LB -- "Auth Requests" --> AuthService
    LB -- "User Requests" --> UserService
    LB -- "Document Requests" --> DocService
    LB -- "WebSocket (Sticky Sessions)" --> CollabService
    LB -- "Distribution Requests" --> DistService
    LB -- "Response Requests" --> RespService
    LB -- "Version Requests" --> DocVersionService
    
    %% Service-to-Service Communication
    AuthService -- "Validate User" --> UserService
    CollabService -- "Apply Operations" --> DocService
    DocVersionService -- "Retrieve Cached Snapshot" --> Redis
    DocVersionService -- "Retrieve Non-Cached Snapshot" --> Mongo
    DocVersionService -- "Send Snapshot" --> APIGW
    DistService -- "Retrieve Document" --> DocService
    DistService -- "Access Form Version" --> DocVersionService
    RespService -- "Validate Form Structure" --> DocService
    
    %% Service to Data Store Communication
    AuthService -- "Verify Credentials" --> PG
    UserService -- "User & Permission Data" --> PG
    DocService -- "Store Documents & Operations" --> Mongo
    DocService -- "Retrieve/Cache Snapshots" --> Redis
    DocService -- "Publish CRDT Operations" --> Kafka
    DocVersionService -- "Retrieve Versions" --> Mongo
    DocVersionService -- "Cache Versions" --> Redis
    RespService -- "Store Responses" --> PG
    
    %% Kafka Consumer Flow
    Kafka -- "CRDT Events" --> KafkaConsumer
    KafkaConsumer -- "Store Version Vectors" --> Mongo
    KafkaConsumer -- "Store Snapshots" --> Mongo
    KafkaConsumer -- "Cache Snapshots" --> Redis
    
    %% Collaboration Service to Clients
    CollabService -- "Broadcast Operations (WebSocket)" --> Client
    
    %% Styling
    classDef client fill:#a3d1ff,stroke:#333,stroke-width:2px;
    classDef gateway fill:#ffbe55,stroke:#333,stroke-width:2px;
    classDef service fill:#c9e793,stroke:#333,stroke-width:2px;
    classDef messaging fill:#ffc6c6,stroke:#333,stroke-width:2px;
    classDef datastore fill:#ffb3e6,stroke:#333,stroke-width:2px;
    
    class Client,Respondent client;
    class APIGW,LB gateway;
    class AuthService,UserService,DocService,DocVersionService,CollabService,DistService,RespService,KafkaConsumer service;
    class Kafka messaging;
    class PG,Mongo,Redis datastore;
