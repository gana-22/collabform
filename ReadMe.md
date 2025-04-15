# CollabForm: A Google Forms-like Product

## Project Overview

This repository contains the system architecture design for CollabForm, a product similar to Google Forms. The system provides an interface for users to create various types of forms (like surveys, quizzes, event registrations), distribute these to respondents via different channels, and collect and analyze responses.

**Note:** Throughout this documentation, I have used "document" as the primary terminology instead of "form" since forms are considered a subset of documents in our architecture approach.

### Key Requirements

- Standalone product, accessible via modern web browsers.
- Capable of User Registration & Authentication, form creation & editing, form distribution, response collection & data analysis.
- Support for real-time collaboration among multiple users.
- Options to customize the look and feel of the documents.
- Privacy and security of document data.
- Scalability and reliability to support a growing user base (potentially millions of users).
- Support for various question types (multiple choice, short answer, rating scale, etc.).

## Technical Implementation Details: Real-time Collaboration

After analyzing both Conflict-free Replicated Data Types (CRDT) and Operational Transformation (OT) approaches for real-time document collaboration, I've chosen to implement **CRDT-based collaboration** for the following reasons:

### Why CRDT Over OT

1. **Decentralized Nature**: CRDT is inherently designed for distributed systems and doesn't require a central server for conflict resolution, making it more resilient to network partitions.

2. **Eventual Consistency**: CRDT guarantees that all replicas will eventually converge to the same state without requiring strict ordering of operations.

3. **Reduced Complexity**: While OT requires complex transformation matrices for different operation types, CRDT's mathematical properties simplify the implementation and reasoning about concurrent edits.

4. **Better Offline Support**: CRDTs handle offline editing and synchronization more naturally, allowing users to continue working even when disconnected.

5. **Scalability Advantages**: CRDTs have better scaling properties for large numbers of concurrent users since they don't need to maintain a global order of operations.

6. **Reduced Server Load**: The conflict resolution in CRDTs happens primarily at the client level, reducing server processing requirements.

- [CRDT Based Sequence Diagram (PNG)](CRDT%20Based%20Sequence%20Diagram.png) - Visual diagram of CRDT approach
- [Detailed System Flow - CRDT Transmission (PDF)](Detailed%20System%20Flow%20%28CRDT%20Transmission%29.pdf) - Detailed CRDT implementation
- [OT Based Flow (PNG)](OT%20Based%20Flow.png) - Alternative OT approach (for reference)

## System Architecture Breakdown

### 1. Domain Model

The domain model describes the entities and entity relationships in the CollabForm system.

- [Domain Model Diagram (PNG)](Q1%3A%20Domain%20Model/Domain%20Model.png) - Visual representation of entities
- [Entity Definitions and Attributes (PDF)](Q1%3A%20Domain%20Model/Domain%20Model.pdf) - Detailed descriptions of each entity

### 2. Data Model and Data Storage

Discusses the document schema structure to support a wide variety of form and question types, and to enable efficient analysis of form responses. This section also covers data consistency approaches across the system.

- [Data Model and Data Storage Documentation (PDF)](Q2%3A%20Data%20Model%20%26%20Data%20Storage/Data%20Model.pdf)

### 3. Business Rules and Constraints

Describes the rules and constraints that govern the behavior of entities, interactions, and processes within the domain.

- [Business Rules and Constraints Documentation (PDF)](Q3%3A%20Business%20Rules%20and%20Constraints/Business%20Rules%20and%20Constraints.pdf)

### 4. High Level System Architecture

An overview of the overall system design architecture.

- [High Level System Architecture Diagram (JPG)](Q4%3A%20High%20Level%20System%20Architecture/High%20Level%20System%20Architecture.jpg)

### 5. Service Level System Design

Details how services communicate with each other and how the client application interacts with the services.

- [Service Level System Design Documentation (PDF)](Q5%3A%20Service%20Level%20System%20Design%20Document/Service%20Level%20System%20Design.pdf)
- [Service Interactions Flow Diagram (PNG)](Q5%3A%20Service%20Level%20System%20Design%20Document/Service%20Level%20System%20Architecture%20Flow.png)

### 6. System Scalability

Details how system design will scale to accommodate increased load, especially during peak usage times and what strategies would be employed to ensure smooth performance even with millions of simultaneous users.

- [System Scalability Documentation (PDF)](Q6%3A%20System%20Scalability/System%20Scalability.pdf)

### 7. Real-Time Collaboration

Details how system handles real-time collaboration on form creation and editing and how system ensures consistency in what users see and experience.

- [Real-Time Collaboration Documentation (PDF)](Q7%3A%20Real-Time%20Collaboration/Real-Time%20Collaboration.pdf)

### 8. Security

Details measures taken to ensure the security of user data and to prevent unauthorized access to forms and responses.
- [Security Documentation (PDF)](Q8%3A%20Security/Security.pdf)

### 9. Failover and Redundancy

Details strategies would be used to ensure high availability of the service and how it handle potential system failures.

- [Failover and Redundancy Documentation (PDF)](Q9%3A%20Failover%20and%20Redundancy/Failover%20and%20Redundancy.pdf)

### 10.1. Process Flows

Visual representations of the process flow within the domain, highlighting the order and conditions under which they occur.

##### a. CRDT Operation Processing Flow
This detailed flow diagram illustrates the process of handling real-time collaborative edits using CRDTs. It showcases:

- Client-side operation generation based on three hybrid event parsing approaches.
- Server-side processing with idempotency checks.
- Distributed lock management for consistency.
- Broadcast to other clients.
- Parallel processing through Kafka.
- Snapshot creation for efficient recovery.

This flow is central to the product's collaborative capabilities and ensures consistent document state across all users.

- [CRDT Operation Processing Flow Diagram (PNG)](Q10%3A%20Process%20Flows%20or%20State%20Transitions/Process%20Flows/CRDT%20Operation%20Processing/CRDT%20Operation%20Processing%20Flow.png)

##### b. Form Response Process Flow
This state diagram shows the journey respondents take when submitting form responses:

- Accessing distribution links.
- Viewing and filling forms.
- Validation of submissions.
- Processing of anonymous vs. identified responses.
- Storage in PostgreSQL with proper relationships.
- Confirmation to respondents.

This process flows provide a comprehensive view of the system's behavior, illustrating how respondents submitting form responses.

- [Form Response Process Flow Diagram (PNG)](Q10%3A%20Process%20Flows%20or%20State%20Transitions/Process%20Flows/Form%20Response/Form%20Response%20Process%20Flow.png)

##### c. Kafka Failure Recovery Process Flow
This process flow diagram illustrates how the system maintains operation during Kafka outages:

- Detection of Kafka unavailability by server components.
- Direct retrieval of pending operations from MongoDB as a fallback.
- Application of operations to in-memory document state.
- Storage of version vectors to maintain consistency.
- Broadcast of CRDT Operations.
- Continuous monitoring for Kafka availability.
- Seamless transition back to normal operation when Kafka recovers.
- Publishing of backlog operations to Kafka upon recovery.
- Processing of operation backlog by Kafka Consumer.

This recovery mechanism ensures the collaborative forms product remains functional even during messaging infrastructure failures, demonstrating the system's resilience and fault tolerance capabilities.

- [Kafka Failure Recovery Process Flow Diagram (PNG)](Q10%3A%20Process%20Flows%20or%20State%20Transitions/Process%20Flows/Kafka%20Failure%20Recovery/Kafka%20Failure%20Recovery%20Process.png)

##### d. Server Recovery Process
This diagram illustrates how the system recovers from server failures:

- Retrieving last processed version vectors.
- Fetching document snapshots.
- Applying pending operations in causal order.
- Validating document consistency.

The diagram demonstrating how the system maintains availability during infrastructure outages.

- [Server Recovery Process Flow Diagram (PNG)](Q10%3A%20Process%20Flows%20or%20State%20Transitions/Process%20Flows/Server%20Recovery/Server%20Recovery%20Process.png)

### 10.2. State Transitions

Visual representations of the possible states and transitions within the domain, highlighting the order and conditions under which they occur.

##### a. Document Lifecycle State Diagram
The document lifecycle diagram shows the primary states a form can transition through, from creation to response collection and closure. This represents the core business process of the product.
Key transitions include:

- Creation of new documents.
- Collaborative editing with CRDT operations.
- Versioning for stable snapshots.
- Distribution to respondents.
- Response collection.
- Closure of forms.

This flow shows how the form transition through to response collection.

- [Document Lifecycle State Diagram (PNG)](Q10%3A%20Process%20Flows%20or%20State%20Transitions/State%20Transitions/Document%20Lifecycle/Document%20Lifecycle%20State%20Diagram.png)

##### b. User Authentication State Diagram
The authentication and authorization flow shows the states and transitions users go through when accessing the system:

- From unauthenticated to submitting credentials.
- JWT verification for subsequent requests.
- Permission checks for resource access.
- Handling of failed authentication or authorization.

This security flow ensures only authorized users can access or modify forms.

- [User Authentication State Diagram (PNG)](Q10%3A%20Process%20Flows%20or%20State%20Transitions/State%20Transitions/User%20Authentication/User%20Authentication%20State%20Diagram.png)

## System Capabilities

The CollabForm product provides:

- User registration with email verification.
- Document creation and editing with multiple question types.
- Document customization (title, description, themes, logo).
- Real-time collaboration features.
- Multiple distribution channels (link sharing, email, embedding).
- Anonymous or identified response collection.
- Real-time response collection and analysis.
- Analytics functionality.

## Non-Functional Requirements

- **Scalability**: Handles high loads and millions of simultaneous users.
- **Reliability**: Includes data backup and recovery measures.
- **Security**: Protects user data and prevents unauthorized access.

## Technology Stack

I've chosen a minimalist approach aligned with existing technology:

- **Frontend/Backend**: JavaScript, Node.js
- **Databases**:
  - MongoDB (document storage)
  - Redis (caching)
- **Message Queue/Broker**:
  - Apache Kafka
- **Collaboration Technology**: CRDT-based implementation

**PostgreSQL** introduced as a new technology following these principles:
- Solves new problems or improves existing solutions with lower resources.
- Open source with free-to-use licensing.
- Actively developed and maintained.

## Implementation Considerations

While designing this system, particular attention was paid to:

1. **Data Consistency**: How to maintain consistency across distributed components.
2. **Scalability**: Approaches to horizontal scaling for peak usage times.
3. **Real-time Collaboration**: Implementation of CRDT for conflict resolution.
4. **Security**: Measures to protect user data and prevent unauthorized access.
5. **High Availability**: Failover and redundancy strategies.