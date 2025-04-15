erDiagram
    user {
        UUID id PK
        String username
        String password_hash
        String email
        Timestamp registration_date
    }

    permission {
        UUID id PK
        UUID user_id FK
        UUID document_id FK
        Enum permission_type
    }

    documents {
        UUID id PK
        String title
        Timestamp creationDate
        Timestamp lastModifiedDate
        UUID ownerUserId FK
        CRDT currentCRDTState
    }

    crdt_operations {
        UUID id PK
        UUID documentId FK
        UUID userId FK
        Enum operationType
        CRDT operationData
        JSON versionVector
        Timestamp timestamp
    }

    document_versions {
        UUID id PK
        UUID documentId FK
        Integer versionNumber
        Timestamp timestamp
        UUID userId FK
        CRDT snapshot
    }

    last_processed_version {
        UUID id PK
        UUID userId FK
        UUID documentId FK
        JSON versionVector
        UUID uniqueId
        Timestamp lastProcessedTimestamp
    }

    document_distributions {
        UUID id PK
        UUID documentVersionId FK
        Enum distributionChannel
        Boolean anonymousResponses
        String distributionLink
    }

    response {
        UUID id PK
        UUID document_version_id FK
        UUID respondent_id FK
        Timestamp submitted_at
        JSON response_data
    }

    respondent_details {
        UUID id PK
        JSON respondent_data
    }

    user ||--o{ documents : creates
    user ||--o{ permission : has
    user ||--o{ crdt_operations : creates
    user ||--o{ document_versions : has
    user ||--o{ last_processed_version : has
    documents ||--o{ crdt_operations : has
    documents ||--o{ document_versions : has
    documents ||--o{ permission : has
    document_versions ||--o{ permission : has
    crdt_operations ||--o{ last_processed_version : has
    document_versions ||--o{ document_distributions : has
    document_versions ||--o{ response : has
    response ||--o{ respondent_details : has
