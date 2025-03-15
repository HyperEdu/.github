# HyperEdu — A High-Performance, Open-Source LMS in Go

**HyperEdu** is an open-source Learning Management System (LMS) platform designed to handle **100,000+ RPS** while remaining **flexible**, **modular**, and **vendor-independent**. Its core is built in **Go** (Golang), leveraging modern architectural patterns:

- **Microservices** and **API-first** design  
- **Event-driven** & **CQRS** approaches  
- **xAPI** integration for detailed analytics (LRS)  
- **Observability** baked in (metrics, tracing, logging)

---

## Goals & Philosophy

1. **High Scalability**: Seamlessly handle large spikes in traffic with horizontal scaling.  
2. **Composable Services**: Each service has a clear domain responsibility (course management, test management, progress tracking, etc.).  
3. **Open Ecosystem**: Provide pluggable modules (e.g., adding a proctoring service, integrating with an existing SSO).  
4. **Community-Driven**: Encouraging open collaboration, sharing best practices for Go, microservices, and high-load systems.

---

## Planned Services / Responsibilities

### 1. Course Management Service
- **Responsibilities**:  
  - CRUD for courses (title, description, modules, content items).  
  - Publish/unpublish courses for learners.  
  - Optionally store references to media files (videos, documents), or integrate with an external storage.
- **Technology**:  
  - Stores core courses in in a PostgreSQL for ACID guarantees with a JSONB column for flexible question formats.
  - Publishes events (like `CourseCreated`, `CourseUpdated`) to an event bus (Kafka, etc.) for read-side or notifications.
  - For Read-Model, consider a separate service or a CQRS pattern. Using a document store (MongoDB) for read-side can be beneficial.

### 2. Test/Quiz Management Service
- **Responsibilities**:  
  - CRUD for tests, question banks, question types (multiple choice, open-ended, etc.).  
  - Logic to assemble sets of questions (e.g., random draws from question banks).  
  - Integrates with a “Test Engine” approach (either in the same service or a separate one) to handle test sessions and scoring.
- **Technology**:  
  - Stores tests and questions in a PostgreSQL for ACID guarantees with a JSONB column for flexible question formats.
  - Emits events upon new/updated tests, or changes in question sets.
  - For read-side, consider a separate service or a CQRS pattern. Using a document store (MongoDB) for read-side can be beneficial.

### 3. Learner Progress & Completion Service
- **Responsibilities**:  
  - Tracks aggregated learner progress across courses, modules, or tasks.  
  - Determines if a user has met completion criteria (e.g., “80% required,” “all tasks done”).  
  - Offers a quick “current status” endpoint (e.g., `GET /progress?courseId=XXX`) without scanning raw logs each time.
- **Internals**:  
  - Subscribes to events like `UserCompletedTask`, `UserScoredTest` from Kafka (or another broker).  
  - Maintains a persistent store of user progress states (in a relational DB or key-value store).

### 4. LRS (Learning Record Store)
- **Responsibilities**:  
  - Receives or subscribes to raw activity statements (`xAPI`) from various services (Test Engine, Course interactions).  
  - Stores them in a time series database (Victoria Metrics, InfluxDB, TimescaleDB) for long-term analytics.
  - Optionally provides an xAPI-compatible API for external tools that require xAPI compliance.
- **Focus**:  
  - High-ingest writes for up to 100k RPS.  
  - Long-term retention for analytics and BI.

### 5. Notification / Event Service
- **Responsibilities**:  
  - Listens for domain events (e.g., “TestPassed,” “CoursePublished”) and triggers notifications (email, WebSocket, push).  
  - Could handle user-specific channels, scheduling, or grouping notifications.
- **Optional**:  
  - Might be part of a generic event subscriber pattern or a stand-alone microservice.

### 6. Identity & Access Management (IAM)
- **Responsibilities**:  
  - User authentication (JWT or OAuth2).  
  - Role-based or permission-based access control (admin, author, learner).  
- **Integration**:  
  - Potential single sign-on (SSO) or external identity providers.  
  - A minimal version can be included just for demonstration.

### 7. Analytics & Reporting Service
- **Responsibilities**:  
  - Aggregates data from the LRS and other services for reporting.  
  - Provides insights on course completion rates, test scores, popular modules, etc.  
  - Optionally integrates with BI tools or data warehouses for advanced analytics.
- **Tech Stack**:
  - Might use a data pipeline (Kafka Streams, Flink) for real-time analytics.  
  - A separate service for complex queries or reports.
  - Use ClickHouse for OLAP queries.

---

## Infrastructure & Observability

- **Containers & Orchestration**:  
  - Kubernetes (Helm charts, Terraform, etc.).  Docker compose for starting development progress.
- **Event Bus**:  
  - Kafka for asynchronous communication and CQRS patterns.  
- **Monitoring**:  
  - Victoria Metrics / Grafana for metrics, Tempo for distributed tracing, and Loki for logging.
- **API Gateway**:  
  - An ingress layer (or L7 router) for a unified endpoint, possible rate-limiting, and auth and scope checks.

---

## Getting Started / Contributing

1. **Clone** the infrastructure repo (e.g., `infra`) and follow instructions to spin up containers (Docker Compose or Helm).  
2. **Run** the core services from the corresponding repos (e.g., `course-service`, `test-service`) in your local environment.  
3. **Explore** the REST/gRPC endpoints (sample Bruno collection or OpenAPI docs may be available).  
4. **Open Issues / PRs**: We welcome feedback, bug reports, or new feature proposals!

---

## Roadmap

Check [ROADMAP.md](./ROADMAP.md) for an iterative plan:

1. **Infra Setup & Docker Compose**  
2. **Basic Course Service**  
3. **CQRS Read-Model**  
4. **Test Service & Question Banks**  
5. **Test Engine (Session/Scoring)**  
6. **LRS & Progress**  
7. **Load Testing & Observability**  
8. Further expansions (Proctoring, Notification, UI Demo, etc.)

We’re excited to build a **high-throughput** LMS that’s flexible enough to adapt to varied e-learning scenarios. Questions and contributions are always welcome — help us shape the future of open-source education platforms!
