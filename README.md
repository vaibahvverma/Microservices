# Go gRPC & GraphQL Microservices Architecture

A production-ready microservices architecture built with Go, utilizing **gRPC** for efficient inter-service communication and a **GraphQL API Gateway** to provide a unified, client-friendly query interface.

---

## Architecture Overview

This project implements an event-driven and domain-driven microservices pattern with isolated data stores:

- **Account Service**: Manages user accounts and authentication profiles (backed by PostgreSQL).
- **Catalog Service**: Handles product management and search indexing (backed by Elasticsearch).
- **Order Service**: Coordinates order creation and tracking (backed by PostgreSQL).
- **GraphQL API Gateway**: Serves as the single entry point for clients, orchestrating downstream gRPC calls to the respective microservices.

```text
                     +-------------------------+
                     |   GraphQL API Gateway   |
                     |     (Port :8000)        |
                     +------------+------------+
                                  |
            +---------------------+---------------------+
            | (gRPC)              | (gRPC)              | (gRPC)
            v                     v                     v
    +---------------+     +---------------+     +---------------+
    |Account Service|     |Catalog Service|     | Order Service |
    |  (Port :8080) |     |  (Port :8080) |     |  (Port :8080) |
    +-------+-------+     +-------+-------+     +-------+-------+
            |                     |                     |
            v                     v                     v
      [PostgreSQL]         [Elasticsearch]        [PostgreSQL]
```

---

## Features

- **Microservices Design**: Clear separation of concerns with domain-specific services.
- **High-Performance RPC**: Inter-service communication powered by gRPC and Protocol Buffers.
- **Unified GraphQL Gateway**: Eliminates over-fetching and under-fetching by exposing flexible GraphQL queries and mutations.
- **Database per Service**: Independent storage mechanisms (PostgreSQL for transactional integrity, Elasticsearch for high-performance catalog search).
- **Dockerized Environment**: Full multi-container orchestration managed via Docker Compose.

---

## Tech Stack

- **Language**: Go
- **API Technologies**: GraphQL, gRPC, Protocol Buffers (protobuf)
- **Databases**: PostgreSQL, Elasticsearch
- **DevOps & Containerization**: Docker, Docker Compose

---

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) & [Docker Compose](https://docs.docker.com/compose/)
- [Go](https://golang.org/doc/install) (for local service development)
- [Protoc](https://github.com/protocolbuffers/protobuf) (if generating protobuf definitions)

### Installation & Running

1. **Clone the repository:**
   ```bash
   git clone https://github.com/vaibahvverma/Microservices.git
   cd Microservices
   ```

2. **Start all services with Docker Compose:**
   ```bash
   docker-compose up -d --build
   ```

3. **Verify the services:**
   Open your browser and navigate to the GraphQL Playground:
   ```text
   http://localhost:8000/playground
   ```

---

## Protocol Buffers Code Generation

If you modify any `.proto` service contracts, regenerate the Go gRPC stubs:

```bash
# Install protoc plugins if not already installed
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Ensure Go bin directory is in your PATH
export PATH="$PATH:$(go env GOPATH)/bin"

# Generate code for account service (example)
protoc --go_out=./account/pb --go-grpc_out=./account/pb ./account/account.proto
```

---

## GraphQL API Examples

Below are sample queries and mutations you can test directly in the GraphQL Playground (`http://localhost:8000/playground`).

### 1. Create an Account
```graphql
mutation {
  createAccount(account: { name: "Alice Smith" }) {
    id
    name
  }
}
```

### 2. Query Accounts
```graphql
query {
  accounts {
    id
    name
  }
}
```

### 3. Create a Product
```graphql
mutation {
  createProduct(
    product: {
      name: "Wireless Mechanical Keyboard"
      description: "Low-profile Bluetooth RGB mechanical keyboard"
      price: 129.99
    }
  ) {
    id
    name
    price
  }
}
```

### 4. Query Products with Pagination & Search
```graphql
query {
  products(pagination: { skip: 0, take: 5 }, query: "Keyboard") {
    id
    name
    description
    price
  }
}
```

### 5. Create an Order
```graphql
mutation {
  createOrder(
    order: {
      accountId: "<ACCOUNT_ID>"
      products: [{ id: "<PRODUCT_ID>", quantity: 2 }]
    }
  ) {
    id
    totalPrice
    products {
      name
      quantity
    }
  }
}
```

### 6. Query Account with Full Order History
```graphql
query {
  accounts(id: "<ACCOUNT_ID>") {
    name
    orders {
      id
      createdAt
      totalPrice
      products {
        name
        quantity
        price
      }
    }
  }
}
```

---

## License

This project is licensed under the [MIT License](LICENSE).