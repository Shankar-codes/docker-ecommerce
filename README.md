# 🛒 docker-ecommerce

A fully containerized, polyglot microservices e-commerce platform orchestrated with **Docker Compose**. Each service is independently built from its own `Dockerfile`, published as a versioned image to Docker Hub under the `shankarntr` namespace, and wired together on a shared Docker bridge network named `ellamma`.

> **Languages:** JavaScript · Java · Python · HTML/CSS · Dockerfile

---

## 🏗️ Architecture Overview

All services communicate over a single custom bridge network (`ellamma`). Only the `frontend` service exposes a port to the host (`80:80`); every other service is reachable only within the Docker network by its container name.

```
                         Host Port 80
                              │
                    ┌─────────▼──────────┐
                    │      frontend       │  (Nginx / HTML + CSS + JS)
                    └──┬──┬──┬──┬────┬───┘
                       │  │  │  │    │
              ┌────────┘  │  │  │    └───────────┐
              ▼           ▼  │  ▼                ▼
         ┌─────────┐  ┌──────┐ ┌────────┐  ┌──────────┐
         │catalogue│  │ user │ │  cart  │  │ shipping │
         └────┬────┘  └──┬───┘ └───┬────┘  └────┬─────┘
              │           │         │             │
              ▼           │    ┌────┤             ▼
         ┌─────────┐      │    │    │        ┌─────────┐
         │ mongodb │      │    ▼    ▼        │  mysql  │
         └─────────┘      │  ┌───────┐       └─────────┘
                          │  │ redis │
                          ▼  └───────┘
                     ┌─────────┐
                     │ mongodb │
                     └─────────┘
                                         ┌──────────┐
                                         │ payment  │──► rabbitmq
                                         └──────────┘
```

---

## 📦 Services

| Service | Docker Image | Language / Runtime | Port | Data Store | Depends On |
|---|---|---|---|---|---|
| `frontend` | `shankarntr/frontend:v1` | HTML, CSS, JavaScript (Nginx) | **80** (host) | — | catalogue, user, cart, shipping, payment |
| `catalogue` | `shankarntr/catalogue:v1` | JavaScript (Node.js) | internal | MongoDB | mongodb |
| `user` | `shankarntr/user:v1` | JavaScript (Node.js) | internal | MongoDB, Redis | redis, mongodb |
| `cart` | `shankarntr/cart:v1` | JavaScript (Node.js) | internal | Redis | redis, catalogue |
| `shipping` | `shankarntr/shipping:v1` | Java (Spring Boot) | internal | MySQL | cart, mysql |
| `payment` | `shankarntr/payment:v1` | Python | internal | RabbitMQ | cart, user, rabbitmq |
| `mongodb` | `shankarntr/mongodb:v1` | MongoDB | internal | Volume: `mongodb` | — |
| `mysql` | `shankarntr/mysql:v1` | MySQL | internal | Volume: `mysql` | — |
| `redis` | `shankarntr/redis:v1` | Redis | internal | Volume: `redis` | — |
| `rabbitmq` | `shankarntr/rabbitmq:v1` | RabbitMQ | internal | Volume: `rabbitmq` | — |

---

## 🗂️ Repository Structure

```
docker-ecommerce/
├── docker-compose.yaml    # Orchestrates all 10 services
│
├── frontend/              # Web UI — HTML, CSS, JS served by Nginx
│   └── Dockerfile
│
├── catalogue/             # Product catalogue service — Node.js + MongoDB
│   └── Dockerfile
│
├── user/                  # User auth & profile service — Node.js
│   └── Dockerfile
│
├── cart/                  # Shopping cart service — Node.js + Redis
│   └── Dockerfile
│
├── shipping/              # Order shipping service — Java (Spring Boot)
│   └── Dockerfile
│
├── payment/               # Payment processing service — Python
│   └── Dockerfile
│
├── mongodb/               # MongoDB image with seed/init scripts
│   └── Dockerfile
│
├── mysql/                 # MySQL image with schema/seed scripts
│   └── Dockerfile
│
├── redis/                 # Redis image (custom config if any)
│   └── Dockerfile
│
└── rabbitmq/              # RabbitMQ image (custom config if any)
    └── Dockerfile
```

---

## 🔧 docker-compose.yaml — Annotated

```yaml
services:

  mongodb:
    build: ./mongodb
    image: shankarntr/mongodb:v1
    volumes:
      - mongodb:/data/db         # Persistent data storage

  catalogue:
    build: ./catalogue
    image: shankarntr/catalogue:v1
    depends_on:
      - mongodb                  # Waits for MongoDB before starting

  redis:
    build: ./redis
    image: shankarntr/redis:v1
    volumes:
      - redis:/data              # Persistent cache storage

  mysql:
    build: ./mysql
    image: shankarntr/mysql:v1
    volumes:
      - mysql:/var/lib/mysql     # Persistent relational data

  rabbitmq:
    build: ./rabbitmq
    image: shankarntr/rabbitmq:v1
    volumes:
      - rabbitmq:/var/lib/rabbitmq  # Persistent message queue

  user:
    build: ./user
    image: shankarntr/user:v1
    depends_on:
      - redis
      - mongodb

  cart:
    build: ./cart
    image: shankarntr/cart:v1
    depends_on:
      - redis
      - catalogue

  shipping:
    build: ./shipping
    image: shankarntr/shipping:v1
    depends_on:
      - cart
      - mysql

  payment:
    build: ./payment
    image: shankarntr/payment:v1
    depends_on:
      - cart
      - user
      - rabbitmq

  frontend:
    build: ./frontend
    image: shankarntr/frontend:v1
    ports:
      - "80:80"                  # Only service exposed to the host
    depends_on:
      - catalogue
      - user
      - cart
      - shipping
      - payment

networks:
  default:
    driver: bridge
    name: ellamma               # All services share this bridge network
    external: false

volumes:
  mongodb:
  redis:
  mysql:
  rabbitmq:
```

---

## 🚀 Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) (v20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (v2.0+ recommended)

### Clone the Repository

```bash
git clone https://github.com/Shankar-codes/docker-ecommerce.git
cd docker-ecommerce
```

### Build and Start All Services

```bash
# Build all images and start all containers in detached mode
docker compose up --build -d
```

This will:
1. Build all 10 Docker images from their respective `Dockerfile`s
2. Create the `ellamma` bridge network
3. Create named volumes for MongoDB, MySQL, Redis, and RabbitMQ
4. Start all containers in dependency order

### Access the Application

Open your browser and navigate to:

```
http://localhost
```

or

```
http://<your-host-ip>
```

### Check Running Services

```bash
# View all running containers and their status
docker compose ps

# Follow logs for all services
docker compose logs -f

# Follow logs for a specific service
docker compose logs -f catalogue
docker compose logs -f shipping
docker compose logs -f payment
```

---

## 🛑 Stopping the Application

```bash
# Stop all containers (preserves volumes and images)
docker compose down

# Stop and remove volumes (destroys all persistent data)
docker compose down -v

# Stop and remove images too
docker compose down --rmi all -v
```

---

## 🔁 Service Startup Order

Docker Compose starts services in dependency order based on `depends_on`. The sequence is:

```
1. mongodb, redis, mysql, rabbitmq   ← data stores (no dependencies)
2. catalogue                          ← depends on mongodb
3. user                               ← depends on redis + mongodb
4. cart                               ← depends on redis + catalogue
5. shipping                           ← depends on cart + mysql
6. payment                            ← depends on cart + user + rabbitmq
7. frontend                           ← depends on all app services
```

> ⚠️ `depends_on` controls **startup order** but does not wait for a service to be *ready*. If a service crashes on first boot due to a dependency not accepting connections yet, use `docker compose restart <service>` or add health checks.

---

## 🐳 Docker Hub Images

All images are published to Docker Hub under the `shankarntr` namespace at version `v1`:

| Image | Docker Hub |
|---|---|
| `shankarntr/frontend:v1` | [hub.docker.com/r/shankarntr/frontend](https://hub.docker.com/r/shankarntr/frontend) |
| `shankarntr/catalogue:v1` | [hub.docker.com/r/shankarntr/catalogue](https://hub.docker.com/r/shankarntr/catalogue) |
| `shankarntr/user:v1` | [hub.docker.com/r/shankarntr/user](https://hub.docker.com/r/shankarntr/user) |
| `shankarntr/cart:v1` | [hub.docker.com/r/shankarntr/cart](https://hub.docker.com/r/shankarntr/cart) |
| `shankarntr/shipping:v1` | [hub.docker.com/r/shankarntr/shipping](https://hub.docker.com/r/shankarntr/shipping) |
| `shankarntr/payment:v1` | [hub.docker.com/r/shankarntr/payment](https://hub.docker.com/r/shankarntr/payment) |
| `shankarntr/mongodb:v1` | [hub.docker.com/r/shankarntr/mongodb](https://hub.docker.com/r/shankarntr/mongodb) |
| `shankarntr/mysql:v1` | [hub.docker.com/r/shankarntr/mysql](https://hub.docker.com/r/shankarntr/mysql) |
| `shankarntr/redis:v1` | [hub.docker.com/r/shankarntr/redis](https://hub.docker.com/r/shankarntr/redis) |
| `shankarntr/rabbitmq:v1` | [hub.docker.com/r/shankarntr/rabbitmq](https://hub.docker.com/r/shankarntr/rabbitmq) |

To run using pre-built images from Docker Hub (without building locally):

```bash
# Pull all images
docker compose pull

# Start without rebuilding
docker compose up -d
```

---

## 🔌 Inter-Service Communication

All services resolve each other by **container name** within the `ellamma` Docker network:

| Source | Target | Purpose |
|---|---|---|
| frontend | catalogue | Fetch product listings |
| frontend | user | Authentication & profile |
| frontend | cart | Add/remove cart items |
| frontend | shipping | Track shipping status |
| frontend | payment | Process payments |
| catalogue | mongodb | Read/write product data |
| user | mongodb | Read/write user profiles |
| user | redis | Session caching |
| cart | redis | Session/cart state |
| cart | catalogue | Fetch product details for cart |
| shipping | mysql | Order and shipping records |
| shipping | cart | Order details |
| payment | rabbitmq | Publish payment events |
| payment | cart | Cart details for payment |
| payment | user | User details for payment |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Orchestration | Docker Compose |
| Networking | Docker Bridge (`ellamma`) |
| Frontend | Nginx, HTML, CSS, JavaScript |
| Catalogue Service | Node.js |
| User Service | Node.js |
| Cart Service | Node.js |
| Shipping Service | Java (Spring Boot) |
| Payment Service | Python |
| Document Database | MongoDB |
| Relational Database | MySQL |
| Cache | Redis |
| Message Broker | RabbitMQ |

---

## 🔗 Related Repositories

| Repository | Purpose |
|---|---|
| **[docker-ecommerce](https://github.com/Shankar-codes/docker-ecommerce)** ← *this repo* | Docker Compose full-stack local deployment |
| **[k8-ecommerce-project](https://github.com/Shankar-codes/k8-ecommerce-project)** | Kubernetes Deployments & Services for the same stack |
| **[k8-ecommerce-database](https://github.com/Shankar-codes/k8-ecommerce-database)** | Kubernetes StatefulSets with persistent AWS EBS storage |
| **[eksctl](https://github.com/Shankar-codes/eksctl)** | AWS EKS cluster provisioning + workstation setup |
| **[k8-resources](https://github.com/Shankar-codes/k8-resources)** | Kubernetes concept reference manifests |

---

## 👤 Author

**Shankar Thimmappa** — DevOps Trainer  
Docker Hub: [hub.docker.com/u/shankarntr](https://hub.docker.com/u/shankarntr)  
GitHub: [@Shankar-codes](https://github.com/Shankar-codes)
