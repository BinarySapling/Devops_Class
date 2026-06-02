# FULL DETAILED SOLUTION: Implementing an API Gateway

---

## 1. Understanding the Scenario

### Real-Life Situation
A microservices application has multiple backend services.

For example:
- one service handles users
- another service handles orders
- another service may handle payments, courses, products, or notifications

If users directly access each service separately, problems can appear:
- many different URLs and ports
- difficult routing
- poor manageability
- weak central control
- harder security handling
- complicated client-side integration

So instead of exposing every service separately, the system uses a **single entry point**.

This single entry point is called an **API Gateway**.

---

## 2. Task Explanation

Students must design a Docker Compose architecture where:

- an **Nginx container** acts as API Gateway
- multiple backend services run behind it
- clients access only the gateway
- the gateway forwards requests to the correct backend service
- the full design is explained line by line

---

## 3. Concepts Covered

This scenario covers:

- API Gateway pattern
- microservices communication
- reverse proxy
- request routing
- Docker Compose orchestration
- service-to-service networking
- single entry point architecture

---

## 4. Core Concept: What is an API Gateway?

An **API Gateway** is a service that sits in front of backend services and acts as the common entry point.

Instead of this:

```text
Client → User Service
Client → Order Service
Client → Payment Service
```

we use this:

```text
Client → API Gateway → User Service
                     → Order Service
                     → Payment Service
```

### Simple meaning
The gateway receives incoming requests and decides where to send them.

---

## 5. Why API Gateway is Useful

An API Gateway provides many benefits:

- single entry point for clients
- hides internal service details
- central request routing
- easier security and authentication integration
- easier logging and monitoring
- easier versioning and policy management
- simpler frontend/client development

In this question, we will focus mainly on:
- routing requests
- reverse proxy behavior
- gateway-based communication

---

## 6. High-Level Architecture

```text
Client / Browser / App
          ↓
      Nginx Gateway
       /        \
      /          \
User Service   Order Service
```

### Explanation
- client sends all requests to Nginx gateway
- Nginx checks request path
- requests for `/users` go to user service
- requests for `/orders` go to order service

---

## 7. Request Routing Example

### Request 1
```text
http://localhost/users
```

### Flow
- request reaches Nginx
- Nginx forwards it to `userservice`

### Request 2
```text
http://localhost/orders
```

### Flow
- request reaches Nginx
- Nginx forwards it to `orderservice`

This is the key idea of API Gateway routing.

---

## 8. Project Structure

A simple demo project may look like this:

```text
api-gateway-demo/
│
├── docker-compose.yml
├── nginx/
│   └── default.conf
├── userservice/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── orderservice/
    ├── Dockerfile
    ├── package.json
    └── server.js
```

### Explanation
- `docker-compose.yml` defines all services
- `nginx/default.conf` contains reverse proxy rules
- `userservice/` contains user microservice
- `orderservice/` contains order microservice

---

# 9. Backend Service 1: User Service

## 9.1 `userservice/package.json`

```json
{
  "name": "userservice",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

## Line-by-line explanation

### Line 1
```json
{
```
Starts JSON object.

**Reasoning:**  
`package.json` must be valid JSON.

### Line 2
```json
"name": "userservice",
```
Defines the project name.

**Reasoning:**  
Helps identify this microservice clearly.

### Line 3
```json
"version": "1.0.0",
```
Defines project version.

**Reasoning:**  
Used for software organization and version tracking.

### Line 4
```json
"main": "server.js",
```
Defines main file.

**Reasoning:**  
Node.js uses this as the entry point.

### Line 5
```json
"scripts": {
```
Starts scripts block.

### Line 6
```json
"start": "node server.js"
```
Defines app start command.

**Reasoning:**  
Container can run service using `npm start`.

### Line 7
```json
},
```
Closes scripts block.

### Line 8
```json
"dependencies": {
```
Starts dependencies section.

### Line 9
```json
"express": "^4.18.2"
```
Adds Express.

**Reasoning:**  
Express is used to create the API service.

### Line 10
```json
}
```
Closes dependencies block.

### Line 11
```json
}
```
Closes JSON object.

---

## 9.2 `userservice/server.js`

```javascript
const express = require("express");

const app = express();
const PORT = 3001;

app.get("/", (req, res) => {
  res.send("Response from User Service");
});

app.listen(PORT, () => {
  console.log(`User Service running on port ${PORT}`);
});
```

---

## Line-by-line explanation

### Line 1
```javascript
const express = require("express");
```
Imports Express.

**Reasoning:**  
Needed to create the HTTP server.

### Line 2
```javascript
const app = express();
```
Creates Express application.

**Reasoning:**  
This object manages routes and request handling.

### Line 3
```javascript
const PORT = 3001;
```
Defines the port for user service.

**Reasoning:**  
Each microservice should run on its own container port.

### Line 4
```javascript
app.get("/", (req, res) => {
```
Defines root route.

**Reasoning:**  
Provides a simple endpoint that can be reached by Nginx.

### Line 5
```javascript
res.send("Response from User Service");
```
Returns service-specific response.

**Reasoning:**  
Helps verify that routing works correctly.

### Line 6
```javascript
});
```
Closes route block.

### Line 7
```javascript
app.listen(PORT, () => {
```
Starts the service.

**Reasoning:**  
Service must listen for requests.

### Line 8
```javascript
console.log(`User Service running on port ${PORT}`);
```
Prints startup message.

**Reasoning:**  
Helps verify startup and debugging.

### Line 9
```javascript
});
```
Closes listen block.

---

## 9.3 `userservice/Dockerfile`

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 3001
CMD ["npm", "start"]
```

---

## Line-by-line explanation

### Line 1
```dockerfile
FROM node:18
```
Uses Node.js base image.

**Reasoning:**  
The service is built in Node.js.

### Line 2
```dockerfile
WORKDIR /app
```
Sets working directory.

**Reasoning:**  
Keeps files organized in the container.

### Line 3
```dockerfile
COPY package.json ./
```
Copies package file.

**Reasoning:**  
Dependencies must be installed inside the image.

### Line 4
```dockerfile
RUN npm install
```
Installs dependencies.

**Reasoning:**  
Express must be present for service execution.

### Line 5
```dockerfile
COPY server.js ./
```
Copies app source code.

**Reasoning:**  
The actual service code must be included.

### Line 6
```dockerfile
EXPOSE 3001
```
Documents service port.

**Reasoning:**  
The user service runs on port 3001.

### Line 7
```dockerfile
CMD ["npm", "start"]
```
Defines startup command.

**Reasoning:**  
Starts user service automatically.

---

# 10. Backend Service 2: Order Service

## 10.1 `orderservice/package.json`

```json
{
  "name": "orderservice",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

## Explanation
This file is structurally the same as the user service package file, but now it belongs to the order service.

**Reasoning:**  
Each microservice needs its own dependency definition and startup metadata.

---

## 10.2 `orderservice/server.js`

```javascript
const express = require("express");

const app = express();
const PORT = 3002;

app.get("/", (req, res) => {
  res.send("Response from Order Service");
});

app.listen(PORT, () => {
  console.log(`Order Service running on port ${PORT}`);
});
```

---

## Line-by-line explanation

### Line 1
```javascript
const express = require("express");
```
Imports Express.

### Line 2
```javascript
const app = express();
```
Creates Express app.

### Line 3
```javascript
const PORT = 3002;
```
Sets port number for order service.

**Reasoning:**  
Order service must run on a different port from user service.

### Line 4
```javascript
app.get("/", (req, res) => {
```
Defines root endpoint.

### Line 5
```javascript
res.send("Response from Order Service");
```
Sends order service response.

**Reasoning:**  
This makes it easy to confirm that Nginx routing is working.

### Line 6
```javascript
});
```
Closes route block.

### Line 7
```javascript
app.listen(PORT, () => {
```
Starts the service.

### Line 8
```javascript
console.log(`Order Service running on port ${PORT}`);
```
Prints startup log.

### Line 9
```javascript
});
```
Closes listen block.

---

## 10.3 `orderservice/Dockerfile`

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 3002
CMD ["npm", "start"]
```

---

## Explanation
This is almost the same as the user service Dockerfile, except the exposed port is 3002.

**Reasoning:**  
Both services are Node.js-based but run independently on different ports.

---

# 11. Nginx API Gateway Configuration

## 11.1 `nginx/default.conf`

```nginx
server {
    listen 80;

    location /users/ {
        proxy_pass http://userservice:3001/;
    }

    location /orders/ {
        proxy_pass http://orderservice:3002/;
    }
}
```

---

# 12. Line-by-Line Explanation of Nginx Configuration

### Line 1
```nginx
server {
```
Starts an Nginx server block.

**Reasoning:**  
This block defines how the gateway handles incoming HTTP requests.

### Line 2
```nginx
listen 80;
```
Tells Nginx to listen on port 80.

**Reasoning:**  
Port 80 is the standard HTTP port.  
The gateway receives client requests on this port.

### Line 3
```nginx
location /users/ {
```
Defines a routing rule for all requests starting with `/users/`.

**Reasoning:**  
Requests coming to this path should go to the user service.

### Line 4
```nginx
proxy_pass http://userservice:3001/;
```
Forwards matching requests to the `userservice` container on port 3001.

**Reasoning:**  
Inside Docker Compose network, the service name `userservice` becomes the hostname.  
So Nginx can reach the container using that name.

### Line 5
```nginx
}
```
Closes the `/users/` location block.

### Line 6
```nginx
location /orders/ {
```
Defines a routing rule for requests starting with `/orders/`.

**Reasoning:**  
These requests should go to the order service.

### Line 7
```nginx
proxy_pass http://orderservice:3002/;
```
Forwards matching requests to the `orderservice` container on port 3002.

**Reasoning:**  
This allows one gateway to route requests to different backend services.

### Line 8
```nginx
}
```
Closes the `/orders/` location block.

### Line 9
```nginx
}
```
Closes the full server block.

---

# 13. Docker Compose File

## 13.1 `docker-compose.yml`

```yaml
services:
  gateway:
    image: nginx:alpine
    container_name: api_gateway
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - userservice
      - orderservice

  userservice:
    build: ./userservice
    container_name: user_service

  orderservice:
    build: ./orderservice
    container_name: order_service
```

---

# 14. Line-by-Line Explanation of Compose File

### Line 1
```yaml
services:
```
Starts services section.

**Reasoning:**  
Docker Compose manages the gateway and backend services together.

### Line 2
```yaml
gateway:
```
Defines the API Gateway service.

**Reasoning:**  
This is the single entry point for users.

### Line 3
```yaml
image: nginx:alpine
```
Uses official lightweight Nginx image.

**Reasoning:**  
Nginx is ideal as a reverse proxy, and Alpine makes the image smaller.

### Line 4
```yaml
container_name: api_gateway
```
Sets container name.

**Reasoning:**  
Makes the gateway easy to identify in Docker commands.

### Line 5
```yaml
ports:
```
Starts gateway port mapping.

### Line 6
```yaml
- "80:80"
```
Maps host port 80 to Nginx container port 80.

**Reasoning:**  
Clients access the whole system through the gateway at `http://localhost`.

### Line 7
```yaml
volumes:
```
Starts volume mount section.

### Line 8
```yaml
- ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
```
Mounts local Nginx config file into container as read-only.

**Reasoning:**  
This allows custom gateway routing rules to be used without rebuilding the Nginx image.

### Line 9
```yaml
depends_on:
```
Starts dependency section.

### Line 10
```yaml
- userservice
```
Gateway depends on user service.

### Line 11
```yaml
- orderservice
```
Gateway depends on order service.

**Reasoning:**  
Gateway is meaningful only if backend services also start.

### Line 12
```yaml
userservice:
```
Defines user service.

### Line 13
```yaml
build: ./userservice
```
Builds user service from local source files.

**Reasoning:**  
User service is a custom application, so it uses `build`.

### Line 14
```yaml
container_name: user_service
```
Sets custom container name.

### Line 15
```yaml
orderservice:
```
Defines order service.

### Line 16
```yaml
build: ./orderservice
```
Builds order service from local source files.

**Reasoning:**  
Order service is also a custom application.

### Line 17
```yaml
container_name: order_service
```
Sets custom container name.

---

# 15. Build vs Image in This Scenario

This question indirectly helps students understand `build` and `image` again.

## `image`
Used for gateway:

```yaml
gateway:
  image: nginx:alpine
```

### Reasoning
Nginx already has an official ready-made image.

## `build`
Used for backend services:

```yaml
userservice:
  build: ./userservice

orderservice:
  build: ./orderservice
```

### Reasoning
These are custom application services, so Docker must build them from local source code.

---

# 16. Full Working Flow

When Docker Compose runs:

1. Nginx image is pulled
2. User service image is built
3. Order service image is built
4. User service container starts
5. Order service container starts
6. Gateway container starts
7. Client accesses `http://localhost/users/`
8. Gateway forwards request to user service
9. Client accesses `http://localhost/orders/`
10. Gateway forwards request to order service

---

# 17. Execution Steps

## Step 1: Create folder structure

```text
api-gateway-demo/
│
├── docker-compose.yml
├── nginx/
│   └── default.conf
├── userservice/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── orderservice/
    ├── Dockerfile
    ├── package.json
    └── server.js
```

---

## Step 2: Write user service files
Use the earlier `package.json`, `server.js`, and `Dockerfile` for `userservice`.

---

## Step 3: Write order service files
Use the earlier `package.json`, `server.js`, and `Dockerfile` for `orderservice`.

---

## Step 4: Write Nginx config
Create `nginx/default.conf` using the earlier reverse proxy configuration.

---

## Step 5: Write `docker-compose.yml`
Use the full Compose file shown earlier.

---

## Step 6: Start the full application

```bash
docker compose up --build
```

### Explanation
- `up` creates and starts services
- `--build` ensures custom backend services are built first

---

## Step 7: Test gateway routes

Open in browser:

### Test User Service
```text
http://localhost/users/
```

Expected output:
```text
Response from User Service
```

### Test Order Service
```text
http://localhost/orders/
```

Expected output:
```text
Response from Order Service
```

---

# 18. Important Networking Concept

Students must understand why these proxy targets work:

```nginx
proxy_pass http://userservice:3001/;
proxy_pass http://orderservice:3002/;
```

### Reason
Docker Compose creates an internal network.  
Inside that network:
- service name becomes hostname
- Nginx can reach `userservice`
- Nginx can reach `orderservice`

So service names act like DNS names inside the Docker Compose environment.

---

# 19. Why API Gateway Pattern is Important in Microservices

Without gateway:
- clients must know many service URLs
- clients become tightly coupled to backend layout
- routing logic is scattered
- security becomes harder to centralize

With gateway:
- one public entry point
- backend services stay internal
- routing is centralized
- microservice communication becomes cleaner
- architecture becomes more manageable

---

# 20. Common Mistakes

## Mistake 1: Exposing all backend services directly
Doing this weakens the single-entry-point design.

**Better approach:**  
Expose only the gateway to users.

---

## Mistake 2: Using `localhost` in `proxy_pass`
Wrong:
```nginx
proxy_pass http://localhost:3001/;
```

### Why wrong?
Inside the gateway container, `localhost` means the gateway container itself, not the backend service container.

Correct:
```nginx
proxy_pass http://userservice:3001/;
```

---

## Mistake 3: Wrong path mapping
If `/users/` is not correctly mapped, requests may not reach the intended service.

---

## Mistake 4: Forgetting to mount Nginx config
Without mounting custom config, Nginx will not know the gateway routing rules.

---

## Mistake 5: Not rebuilding after changes
If backend code changes, run:

```bash
docker compose up --build
```

---

# 21. Optional Advanced Improvements

In real systems, an API Gateway may also handle:
- authentication
- authorization
- rate limiting
- SSL termination
- caching
- request logging
- response transformation
- API versioning

For this classroom question, only reverse proxy routing is necessary, but students should know the API Gateway can do much more.

---

# 22. Complete Final Files

## File 1: `userservice/package.json`

```json
{
  "name": "userservice",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

## File 2: `userservice/server.js`

```javascript
const express = require("express");

const app = express();
const PORT = 3001;

app.get("/", (req, res) => {
  res.send("Response from User Service");
});

app.listen(PORT, () => {
  console.log(`User Service running on port ${PORT}`);
});
```

---

## File 3: `userservice/Dockerfile`

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 3001
CMD ["npm", "start"]
```

---

## File 4: `orderservice/package.json`

```json
{
  "name": "orderservice",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

## File 5: `orderservice/server.js`

```javascript
const express = require("express");

const app = express();
const PORT = 3002;

app.get("/", (req, res) => {
  res.send("Response from Order Service");
});

app.listen(PORT, () => {
  console.log(`Order Service running on port ${PORT}`);
});
```

---

## File 6: `orderservice/Dockerfile`

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 3002
CMD ["npm", "start"]
```

---

## File 7: `nginx/default.conf`

```nginx
server {
    listen 80;

    location /users/ {
        proxy_pass http://userservice:3001/;
    }

    location /orders/ {
        proxy_pass http://orderservice:3002/;
    }
}
```

---

## File 8: `docker-compose.yml`

```yaml
services:
  gateway:
    image: nginx:alpine
    container_name: api_gateway
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - userservice
      - orderservice

  userservice:
    build: ./userservice
    container_name: user_service

  orderservice:
    build: ./orderservice
    container_name: order_service
```

---

# 23. Viva / Exam Explanation

A student can explain the solution like this:

> In a microservices system, multiple backend services should not always be directly exposed to users. An API Gateway pattern provides a single entry point for all client requests. In this solution, an Nginx container is used as a reverse proxy gateway. Requests to `/users/` are routed to the user service, and requests to `/orders/` are routed to the order service. Docker Compose is used to run the gateway and backend services together. The gateway communicates with backend services using service names over the internal Docker Compose network.

---

# 24. Short Answer Version

> An API Gateway is used in microservices to provide a single entry point for clients. In this solution, Nginx acts as a reverse proxy gateway. It routes requests to different backend services based on URL paths, such as `/users/` for the user service and `/orders/` for the order service. Docker Compose is used to deploy the gateway and services together, and service names are used for internal communication.

---

# 25. Final Conclusion

This scenario teaches a very important microservices design pattern:

**clients should access multiple services through one controlled gateway.**

The correct solution uses:
- Nginx as API Gateway
- Docker Compose for orchestration
- custom backend services
- reverse proxy routing rules
- service-name-based internal communication

So the final architecture provides:
- a single public entry point
- centralized routing
- hidden internal services
- cleaner microservices communication
- better backend structure for future growth

