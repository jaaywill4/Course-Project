# Improving Scalability and Load Balancing of a Containerized Cloud Application

## Vision
This project demonstrates a two-component containerized application using Infrastructure as Code principles.

### Architecture

![Architecture Diagram](docs/architecture.png)

### Components

Component 1: Web Application (Node.js)
- Handles HTTP requests from users.
- Displays the current visitor count.
- Increments the visitor count on each request.

Component 2: Redis
- Stores the visitor counter.
- Updates the count when requested by the web application.
- Communicates with the web application using TCP.

### Communication Flow

Browser -> HTTP -> Web App
Web App -> TCP (Redis protocol) -> Redis
Redis -> Web Application -> Browser


## Proposal

This project will use lightweight container images for deployment.

- Web Application Base Image: `node:18-alpine`
- Redis Base Image: `redis:7-alpine`

The final deployment will be demonstrated on CloudLab using containerization.

## Build Process

I created a Dockerfile inside the app folder to containerize my Node.js application.
Here is what each line does:

- FROM node:18-alpine
    I chose this base image because it is a smaller version of Node.js, which makes the container lighter and faster to run.

- WORKDIR /app
    This sets the working directory inside the container so all commands run in the app folder.

- COPY package*.json ./
    This copies the package files first so Docker can cache the dependencies and not reinstall them every time.

- RUN npm install
    This installs all the dependencies needed for the app like express and redis.

- COPY . .
  This copies the rest of the application code into the container.

- EXPOSE 3000
  This tells Docker that the app will run on port 3000.

- CMD ["node", "app.js"]
    This starts the Node.js application when the container runs.
I used docker-compose to build and run both the web container and the Redis container together.

## Networking

The containers communicate using Docker's default bridge network.
When I run docker-compose, Docker automatically puts both containers on the same network and assigns them IP addresses.
Instead of using IP addresses directly, I used the name "redis" in my code to connect to the Redis container. Docker automatically translates this name to the correct IP address.
This makes it easier because I don't have to worry about IPs changing.

The communcation works like this:
1. The browser sends a request to the web container
2. The web container connects to Redis using TCP on port 6379
3. Redis updates the visitor count and sends it back.

This setup allows the containers to talk to each other internally without exposing Redis to the outside.

### System Overview

This project implements a cloud-native two-tier application using Docker containers.
The system consists of a Node.js wrb server and a Redis database that work together to maintain a visitor count.
When a user accesses the application, the web server increments a counter stored in Redis and displays the updated value.

### System Design

The architecture follows a simplle client-server model:
- Client sends HTTP request to the Node.js container
- Node.js container communicates with Redis container
- Redis stores the visitor count

This separation of services demonstrates a real-world microservice pattern.

### Deployment Process

The system is deployed using Docker Compose.

Steps:
1. Build the Node.js container using the Dockerfile
2. Pull the Redis image from Docker Hub
3. Start both containers using:
   docker compose up --build

Docker Compose automatically creates a network and connects the services.

### Key Features

- Multi-container architecture
- Custom Dockerfile for Node.js application
- Redis integration for persistent counting
- Container networking using service names
- Graceful handling of Redis connection issues

### Challenges and Solutions 

**Challenge:** Redis may not be ready when the web server starts
**Solution:** Used Docker Compose networking and service name resolution

### Future Improvements

- Add a fronted UI
- Implement authentication
- Deploy using CI/CD pipeline
- Add container security features (non-root user, resource limits)

## Milestone 2 — Kubernetes Core Implementation

### Overview

For Milestone 2, I moved my Node.js visitor counter application from Docker Compose to a Kubernetes cluster running on FABRIC. The goal was to improve scalability and reliability by using Kubernetes Deployments, Services, and multiple application replicas.

### Kubernetes Architecture

The application uses three main components:

- **Nginx:** Receives incoming HTTP requests and forwards them to the Node.js Service.
- **Node.js:** Runs three replicas of my visitor counter application.
- **Redis:** Stores the visitor count shared by all three Node.js replicas.

I used ClusterIP Services for internal communication and a NodePort Service to expose Nginx on port 30081.

### Deployment Instructions

The Kubernetes configuration files are organized into three folders: `configmaps/`, `deployments/`, and `services/`.

To deploy the application, run these commands from the project directory on a machine configured to access the Kubernetes cluster:

```bash
kubectl apply -f configmaps/
kubectl apply -f deployments/
kubectl apply -f services/
```

To verify the deployment:

```bash
kubectl get deploy,pods,svc
```

### Accessing the Application

The application can be accessed through the Nginx NodePort Service on port 30081.

For testing from FABRIC node1, I used:

```bash
curl http://192.168.1.1:30081
```

The application returned the visitor count, which increased with each request.

### Scaling Test

I tested scaling by increasing the Node.js Deployment from three replicas to five.

```bash
kubectl scale deployment course-project-nodejs --replicas=5
kubectl get pods -l app=course-project-nodejs
```

All five Pods reached `2/2 Running`. After testing, I scaled the Deployment back to three replicas.

### Self-Healing Test

I manually deleted one Node.js Pod to test Kubernetes self-healing. Kubernetes automatically created a replacement Pod, and the Deployment returned to three running replicas.

### Multi-Container Pod

Each Node.js Pod contains two containers: the Node.js application and a BusyBox logging sidecar.

The sidecar prints a status message every ten seconds. I added it to demonstrate how multiple containers can run together inside the same Pod. It currently provides basic status logging rather than collecting application access logs.

### Persistence

Redis currently uses temporary storage through an `emptyDir` volume. Persistent storage will be added in Milestone 3.

### Milestone 2 Results

I successfully deployed the application on Kubernetes, tested communication between Nginx, Node.js, and Redis, scaled the application from three to five replicas, verified automatic Pod replacement, and confirmed that the sidecar generated logs.
