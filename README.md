# Ryde - Distributed Ride-Sharing Platform

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Technology Stack](#-technology-stack)
- [Prerequisites](#-prerequisites)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Services](#-services)
- [Development](#-development)
- [Infrastructure](#-infrastructure)
- [Observability](#-observability)
- [Contributing](#-contributing)

---

## 🚀 Overview

**Ryde** is a modern, distributed ride-sharing platform designed to demonstrate production-grade microservices architecture. The system handles real-time driver matching, trip management, payment processing, and live location tracking.

### Key Features

- 🗺️ **Real-time Location Tracking** - Live driver locations using geohashing
- 🚗 **Smart Driver Matching** - Efficient algorithm to match riders with nearby drivers
- 💳 **Secure Payments** - Integrated Stripe payment processing
- 📡 **WebSocket Communication** - Real-time updates for riders and drivers
- 🔍 **Distributed Tracing** - Full observability with Jaeger and OpenTelemetry
- 📨 **Event-Driven Architecture** - Async communication via RabbitMQ
- ⚡ **High Performance** - gRPC for inter-service communication
- 🐳 **Cloud Native** - Kubernetes-ready with Docker containerization

---

## 🏗️ Architecture

Ryde follows a **microservices architecture** with event-driven communication patterns:

```
┌─────────────┐
│   Web App   │ (Next.js)
└──────┬──────┘
       │ HTTP/WS
       ▼
┌─────────────────┐
│  API Gateway    │ (HTTP/WebSocket → gRPC)
└────────┬────────┘
         │ gRPC
    ┌────┴────┬─────────────┐
    ▼         ▼             ▼
┌─────────┐ ┌──────────┐ ┌─────────┐
│  Trip   │ │  Driver  │ │ Payment │
│ Service │ │ Service  │ │ Service │
└────┬────┘ └────┬─────┘ └────┬────┘
     │           │            │
     └───────────┴────────────┘
              │
         ┌────▼────┐
         │RabbitMQ │ (Event Bus)
         └─────────┘
              │
         ┌────▼────┐
         │ MongoDB │ (Database)
         └─────────┘
```

### Communication Patterns

- **External → API Gateway**: REST APIs & WebSockets
- **API Gateway → Services**: gRPC (synchronous)
- **Service → Service**: RabbitMQ (asynchronous events)
- **All Services**: MongoDB for persistence

---

## 🛠️ Technology Stack

### Backend

- **Language**: Go 1.23.0
- **RPC**: gRPC with Protocol Buffers
- **Message Queue**: RabbitMQ (AMQP 0.9.1)
- **Database**: MongoDB
- **Payments**: Stripe API
- **Tracing**: Jaeger + OpenTelemetry

### Frontend

- **Framework**: Next.js 15.1.5 (React 19)
- **UI**: TailwindCSS + Radix UI
- **Maps**: Leaflet + React Leaflet
- **Real-time**: WebSockets
- **TypeScript**: Full type safety

### Infrastructure

- **Orchestration**: Kubernetes
- **Containerization**: Docker
- **Local Dev**: Tilt (hot-reload enabled)
- **CI/CD**: Production & development environments

---

## 📦 Prerequisites

Before you begin, ensure you have the following installed:

- **Go** 1.23.0 or higher ([Download](https://golang.org/dl/))
- **Node.js** 20+ and npm ([Download](https://nodejs.org/))
- **Docker** and Docker Compose ([Download](https://www.docker.com/))
- **Kubernetes** (Docker Desktop, Minikube, or Kind)
- **Tilt** ([Install](https://docs.tilt.dev/install.html))
- **kubectl** ([Install](https://kubernetes.io/docs/tasks/tools/))
- **Protocol Buffers Compiler** (protoc) ([Install](https://grpc.io/docs/protoc-installation/))

### Optional Tools

- **MongoDB Compass** - Database GUI
- **Postman** - API testing
- **k9s** - Kubernetes CLI manager

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/ryde.git
cd ryde
```

### 2. Configure Secrets

Create your secrets file from the template:

```bash
cp infra/development/k8s/secrets.template.yaml infra/development/k8s/secrets.yaml
```

Edit `secrets.yaml` with your actual values:

- MongoDB connection string
- Stripe API keys
- RabbitMQ credentials (if different from defaults)

### 3. Install Go Dependencies

```bash
go mod download
```

### 4. Generate Protocol Buffers

```bash
make generate-proto
```

### 5. Install Frontend Dependencies

```bash
cd web
npm install
cd ..
```

### 6. Start the Application with Tilt

Ensure your Kubernetes cluster is running, then:

```bash
tilt up
```

This will:

- Build all Docker images
- Deploy services to Kubernetes
- Enable hot-reloading for rapid development
- Open Tilt UI in your browser

### 7. Access the Application

Once all services are running:

| Service         | URL                    | Description                           |
| --------------- | ---------------------- | ------------------------------------- |
| **Web App**     | http://localhost:3000  | Main user interface                   |
| **API Gateway** | http://localhost:8081  | REST API & WebSocket                  |
| **RabbitMQ UI** | http://localhost:15672 | Message queue dashboard (guest/guest) |
| **Jaeger UI**   | http://localhost:16686 | Distributed tracing                   |
| **Tilt UI**     | http://localhost:10350 | Development dashboard                 |

---

## 📁 Project Structure

```
ryde/
├── services/                 # Microservices
│   ├── api-gateway/         # HTTP/WS entry point
│   ├── trip-service/        # Trip management
│   ├── driver-service/      # Driver management
│   └── payment-service/     # Payment processing
├── shared/                   # Shared libraries
│   ├── contracts/           # API contracts
│   ├── messaging/           # RabbitMQ utilities
│   ├── proto/               # Generated protobuf code
│   ├── tracing/             # OpenTelemetry setup
│   └── types/               # Common types
├── web/                      # Next.js frontend
│   ├── src/
│   │   ├── app/            # App router pages
│   │   ├── components/     # React components
│   │   ├── hooks/          # Custom hooks
│   │   └── utils/          # Utility functions
│   └── package.json
├── proto/                    # Protocol buffer definitions
├── infra/                    # Infrastructure configs
│   ├── development/         # Local dev setup
│   │   ├── docker/         # Dockerfiles
│   │   └── k8s/            # Kubernetes manifests
│   └── production/          # Production configs
├── docs/                     # Documentation
│   └── architecture/        # Architecture diagrams
├── tools/                    # Development tools
├── Tiltfile                  # Tilt configuration
├── Makefile                  # Build commands
└── go.mod                    # Go dependencies
```

---

## 🔧 Services

### API Gateway

**Port**: 8081 | **Protocol**: HTTP, WebSocket → gRPC

- Entry point for all external requests
- REST API endpoints for riders and drivers
- WebSocket connections for real-time updates
- Routes requests to appropriate microservices via gRPC
- JWT authentication middleware

**Key Endpoints**:

- `POST /api/trips` - Create a new trip
- `GET /api/trips/:id` - Get trip details
- `WS /ws/rider` - Rider WebSocket connection
- `WS /ws/driver` - Driver WebSocket connection

### Trip Service

**Protocol**: gRPC, RabbitMQ

Core business logic for trip management:

- Trip creation and lifecycle management
- Dynamic fare calculation
- Trip state transitions (requested → matched → started → completed)
- Publishes trip events to RabbitMQ
- Consumes driver and payment events

**Database**: MongoDB (`trips` collection)

### Driver Service

**Protocol**: gRPC, RabbitMQ

Manages driver operations:

- Driver location updates (geohash-based)
- Driver availability status
- Finds nearby available drivers
- Processes trip assignment events
- Updates driver state (available/busy)

**Database**: In-memory store with persistence option

### Payment Service

**Protocol**: RabbitMQ

Handles payment processing:

- Stripe integration for payment processing
- Listens to trip completion events
- Creates payment intents
- Processes charges
- Publishes payment status events

**Integration**: Stripe API

---

## 💻 Development

### Hot Reloading

Tilt provides automatic hot-reloading:

- Backend services rebuild on Go file changes
- Frontend hot-reloads on TypeScript/React changes
- No manual restart needed

### Building Services Manually

```bash
# Build all services
go build ./services/api-gateway
go build ./services/trip-service/cmd/main.go
go build ./services/driver-service
go build ./services/payment-service/cmd/main.go

# Build frontend
cd web && npm run build
```

### Running Tests

```bash
# Run Go tests
go test ./...

# Run frontend tests
cd web && npm test
```

### Generating Protocol Buffers

After modifying `.proto` files:

```bash
make generate-proto
```

### Adding a New Service

Use the service generator tool:

```bash
go run tools/create_service.go <service-name>
```

---

## 🏢 Infrastructure

### Kubernetes Manifests

Each service has:

- **Deployment**: Manages pods and replicas
- **Service**: Internal networking
- **ConfigMap**: Environment configuration
- **Secrets**: Sensitive data

### Environment Configuration

Two environments available:

- **Development** (`infra/development/`): Hot-reload, debug settings
- **Production** (`infra/production/`): Optimized, secure, scalable

### Deployment

#### Development (Local)

```bash
tilt up
```

#### Production

```bash
# Apply production configs
kubectl apply -f infra/production/k8s/

# Or use your CI/CD pipeline
```

---

## 📊 Observability

### Distributed Tracing

**Jaeger** provides end-to-end request tracing:

- Trace requests across all microservices
- Identify performance bottlenecks
- Debug distributed transactions

Access Jaeger UI: http://localhost:16686

### Logging

All services use structured logging:

- Service name
- Trace ID / Span ID correlation
- Request/Response logging

### Monitoring

The system includes:

- **OpenTelemetry** instrumentation on all services
- **gRPC interceptors** for trace propagation
- **HTTP middleware** for request tracking
- **RabbitMQ tracing** for async operations

---

## 📚 Documentation

Additional documentation available in the `docs/` directory:

- [Architecture Diagrams](docs/architecture/)
- [RabbitMQ Flow](docs/architecture/rabbitmq-flow-v1.md)
- [Trip Creation Flow](docs/architecture/trip-creation-flow-v1.md)
