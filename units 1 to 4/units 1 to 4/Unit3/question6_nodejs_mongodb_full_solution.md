# FULL DETAILED SOLUTION: Deploying Node.js + MongoDB Application

---

## 1. Understanding the Scenario

### Real-Life Situation
A company has a **Node.js API service** that performs business logic and handles user requests.

This API service needs a backend database to:
- store records
- retrieve data
- update documents
- delete data when required

The database chosen is **MongoDB**.

So the full application needs two components:

- **Node.js application service**
- **MongoDB database service**

This makes it a **multi-container deployment** because the application is made of more than one container.

---

## 2. Task Explanation

Students must:

- create a Docker Compose setup
- deploy the Node.js application service
- deploy the MongoDB service
- connect Node.js to MongoDB properly
- understand the use of `build` and `image`
- explain the whole implementation line by line
- run the application using Docker Compose

---

## 3. Concepts Covered

This question covers:

- build vs image fields
- microservices deployment
- Docker Compose orchestration
- service-to-service communication
- environment variable configuration
- containerized Node.js deployment
- MongoDB integration

---

## 4. Core Concept: Why Two Containers?

A company API and database should usually run separately because:

- API and database have different roles
- each service can be managed independently
- easier scaling
- better isolation
- simpler maintenance
- closer to real-world DevOps practice

### Role of each service

#### Node.js container
- runs backend API
- handles HTTP requests
- connects to database
- processes application logic

#### MongoDB container
- stores data as documents
- provides persistent backend storage
- serves database operations to the API

---

## 5. High-Level Architecture

```text
Client / Browser / API Tester
            ↓
      Node.js API Service
            ↓
        MongoDB Service
```

### Explanation
- user or client sends request to Node.js API
- Node.js processes request
- Node.js stores or reads data from MongoDB
- MongoDB returns results
- Node.js sends response back to client

---

## 6. Project Structure

A simple project structure can be:

```text
node-mongo-demo/
│
├── docker-compose.yml
└── app/
    ├── Dockerfile
    ├── package.json
    └── server.js
```

### Explanation
- `docker-compose.yml` defines both services
- `app/` contains custom Node.js application code
- MongoDB uses official image, so it does not need custom Dockerfile in this basic example

---

# 7. Full Implementation

We will use:

- Node.js
- Express
- Mongoose
- MongoDB
- Docker Compose

This creates a realistic but beginner-friendly example.

---

## 7.1 `app/package.json`

```json
{
  "name": "node-mongo-app",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.6.0"
  }
}
```

---

## Line-by-line explanation of `package.json`

### Line 1
```json
{
```
Starts the JSON object.

**Reasoning:**  
`package.json` must follow proper JSON structure.

### Line 2
```json
"name": "node-mongo-app",
```
Defines the application name.

**Reasoning:**  
Helps identify the project.

### Line 3
```json
"version": "1.0.0",
```
Defines current version.

**Reasoning:**  
Useful for software organization and versioning.

### Line 4
```json
"main": "server.js",
```
Defines main application file.

**Reasoning:**  
Node.js knows the primary entry point of the project.

### Line 5
```json
"scripts": {
```
Starts scripts section.

**Reasoning:**  
Allows commands like `npm start`.

### Line 6
```json
"start": "node server.js"
```
Defines application start command.

**Reasoning:**  
Container can run app easily using standard npm command.

### Line 7
```json
},
```
Closes scripts block.

### Line 8
```json
"dependencies": {
```
Starts dependencies block.

**Reasoning:**  
Dependencies are required external libraries.

### Line 9
```json
"express": "^4.18.2",
```
Adds Express.

**Reasoning:**  
Used to create REST API and server routes.

### Line 10
```json
"mongoose": "^7.6.0"
```
Adds Mongoose.

**Reasoning:**  
Mongoose is used to connect Node.js application to MongoDB.

### Line 11
```json
}
```
Closes dependencies block.

### Line 12
```json
}
```
Closes JSON object.

---

## 7.2 `app/server.js`

```javascript
const express = require("express");
const mongoose = require("mongoose");

const app = express();
const PORT = 3000;

// MongoDB connection URI
const MONGO_URI = "mongodb://mongodb:27017/companydb";

// Connect to MongoDB
mongoose.connect(MONGO_URI)
  .then(() => console.log("Connected to MongoDB successfully"))
  .catch((err) => console.log("MongoDB connection error:", err.message));

app.get("/", (req, res) => {
  res.send("Node.js API is running with MongoDB backend");
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

---

## Line-by-line explanation of `server.js`

### Line 1
```javascript
const express = require("express");
```
Imports Express library.

**Reasoning:**  
Express is needed to create the Node.js API server.

### Line 2
```javascript
const mongoose = require("mongoose");
```
Imports Mongoose library.

**Reasoning:**  
Mongoose is used for MongoDB connection and data modeling.

### Line 3
```javascript
const app = express();
```
Creates Express application object.

**Reasoning:**  
This object manages routes and server behavior.

### Line 4
```javascript
const PORT = 3000;
```
Stores server port.

**Reasoning:**  
Keeps configuration readable and easy to change.

### Line 5
```javascript
const MONGO_URI = "mongodb://mongodb:27017/companydb";
```
Defines the MongoDB connection URI.

**Reasoning:**  
- `mongodb` is the service name of MongoDB container
- `27017` is the default MongoDB port
- `companydb` is the database name

Docker Compose networking allows Node.js to reach MongoDB using service name as hostname.

### Line 6
```javascript
mongoose.connect(MONGO_URI)
```
Starts MongoDB connection.

**Reasoning:**  
The app must connect to database before doing real work.

### Line 7
```javascript
.then(() => console.log("Connected to MongoDB successfully"))
```
Runs on successful connection.

**Reasoning:**  
Useful for verifying that service-to-service communication works.

### Line 8
```javascript
.catch((err) => console.log("MongoDB connection error:", err.message));
```
Runs on connection failure.

**Reasoning:**  
Helps identify database connection issues.

### Line 9
```javascript
app.get("/", (req, res) => {
```
Defines root API route.

**Reasoning:**  
Allows testing the API easily.

### Line 10
```javascript
res.send("Node.js API is running with MongoDB backend");
```
Sends simple response.

**Reasoning:**  
Confirms that the application is running.

### Line 11
```javascript
});
```
Closes route block.

### Line 12
```javascript
app.listen(PORT, () => {
```
Starts server listening on port 3000.

**Reasoning:**  
This makes the API accessible from outside the container.

### Line 13
```javascript
console.log(`Server is running on port ${PORT}`);
```
Prints startup message.

**Reasoning:**  
Helps confirm successful startup.

### Line 14
```javascript
});
```
Closes listen block.

---

## 7.3 `app/Dockerfile`

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 3000
CMD ["npm", "start"]
```

---

## Line-by-line explanation of `Dockerfile`

### Line 1
```dockerfile
FROM node:18
```
Uses Node.js 18 base image.

**Reasoning:**  
The application is written in Node.js, so it needs a Node runtime.

### Line 2
```dockerfile
WORKDIR /app
```
Sets working directory inside container.

**Reasoning:**  
Keeps app files organized.

### Line 3
```dockerfile
COPY package.json ./
```
Copies package file into container.

**Reasoning:**  
Dependencies must be installed inside the image.

### Line 4
```dockerfile
RUN npm install
```
Installs required packages.

**Reasoning:**  
Express and Mongoose are necessary for app execution.

### Line 5
```dockerfile
COPY server.js ./
```
Copies application source code.

**Reasoning:**  
The container needs the Node.js program file.

### Line 6
```dockerfile
EXPOSE 3000
```
Documents that app listens on port 3000.

**Reasoning:**  
Makes intended application port explicit.

### Line 7
```dockerfile
CMD ["npm", "start"]
```
Defines default container startup command.

**Reasoning:**  
Automatically starts the Node.js application when the container runs.

---

# 8. Docker Compose Solution

## 8.1 `docker-compose.yml`

```yaml
services:
  nodeapp:
    build: ./app
    container_name: node_api_app
    ports:
      - "3000:3000"
    depends_on:
      - mongodb

  mongodb:
    image: mongo:6
    container_name: mongodb_service
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

---

# 9. Line-by-Line Explanation of `docker-compose.yml`

### Line 1
```yaml
services:
```
Starts services section.

**Reasoning:**  
Docker Compose manages multiple containers as services.

### Line 2
```yaml
nodeapp:
```
Defines Node.js application service.

**Reasoning:**  
This is the custom backend API container.

### Line 3
```yaml
build: ./app
```
Builds the Node.js image from the local `app` folder.

**Reasoning:**  
This service uses custom application code, so Docker must build an image from source files.

### Line 4
```yaml
container_name: node_api_app
```
Sets custom container name.

**Reasoning:**  
Makes the running container easier to identify.

### Line 5
```yaml
ports:
```
Starts port mapping section.

**Reasoning:**  
The API must be accessible from the host machine.

### Line 6
```yaml
- "3000:3000"
```
Maps host port 3000 to container port 3000.

**Reasoning:**  
Allows browser or API client to access Node.js service.

### Line 7
```yaml
depends_on:
```
Starts dependency definition.

**Reasoning:**  
Node.js depends on MongoDB.

### Line 8
```yaml
- mongodb
```
Specifies that Node.js depends on MongoDB service.

**Reasoning:**  
Compose starts MongoDB before Node.js.

### Line 9
```yaml
mongodb:
```
Defines MongoDB service.

**Reasoning:**  
This is the database container.

### Line 10
```yaml
image: mongo:6
```
Uses official MongoDB image version 6.

**Reasoning:**  
Since MongoDB does not need custom code here, we directly use a ready-made image.

### Line 11
```yaml
container_name: mongodb_service
```
Sets custom name for MongoDB container.

**Reasoning:**  
Helps in container management and debugging.

### Line 12
```yaml
ports:
```
Starts database port mapping.

**Reasoning:**  
Useful for direct testing or inspection if needed.

### Line 13
```yaml
- "27017:27017"
```
Maps MongoDB port.

**Reasoning:**  
27017 is default MongoDB port.

### Line 14
```yaml
volumes:
```
Starts volume section for MongoDB.

**Reasoning:**  
Database data should persist beyond container life.

### Line 15
```yaml
- mongo_data:/data/db
```
Mounts named volume into MongoDB data directory.

**Reasoning:**  
MongoDB stores data in `/data/db`, so this ensures persistence.

### Line 16
```yaml
volumes:
```
Starts top-level volume definitions.

**Reasoning:**  
Compose must know which named volumes to create.

### Line 17
```yaml
mongo_data:
```
Defines the named volume.

**Reasoning:**  
Used to persist MongoDB data.

---

# 10. Build vs Image: Very Important Concept

This question specifically covers **build vs image fields**, so students must understand the difference clearly.

## `build`
Used when:
- we have custom source code
- we have our own Dockerfile
- we want Docker to create image locally

Example:
```yaml
nodeapp:
  build: ./app
```

### Reasoning
The Node.js application is custom.  
We wrote:
- `server.js`
- `package.json`
- `Dockerfile`

So Docker must build an image from these files.

---

## `image`
Used when:
- we want to use a ready-made image
- no custom application code is needed
- official image is enough

Example:
```yaml
mongodb:
  image: mongo:6
```

### Reasoning
MongoDB already has an official ready-to-use image, so there is no need to build one ourselves.

---

## Simple comparison

### `build`
- creates image from local Dockerfile
- used for custom applications
- useful when writing our own code

### `image`
- pulls ready-made image
- used for standard services
- useful for official software like MongoDB, MySQL, Redis, Nginx

---

# 11. Full Working Flow

When we run Docker Compose:

1. Docker reads `docker-compose.yml`
2. It builds Node.js image from `./app`
3. It pulls MongoDB image from Docker Hub
4. MongoDB container starts
5. Node.js container starts
6. Node.js tries to connect to MongoDB
7. API becomes available on port 3000
8. User accesses the application

---

# 12. Execution Steps

## Step 1: Create folder structure

```text
node-mongo-demo/
│
├── docker-compose.yml
└── app/
    ├── Dockerfile
    ├── package.json
    └── server.js
```

---

## Step 2: Write `package.json`
Use the earlier backend package file.

---

## Step 3: Write `server.js`
Use the earlier server file.

---

## Step 4: Write `Dockerfile`
Use the earlier Node.js Dockerfile.

---

## Step 5: Write `docker-compose.yml`
Use the full Compose file shown earlier.

---

## Step 6: Start the application

```bash
docker compose up --build
```

### Explanation
- `up` starts services
- `--build` rebuilds images if needed

---

## Step 7: Check running containers

```bash
docker ps
```

### Expected
You should see:
- Node.js container
- MongoDB container

---

## Step 8: Open browser or API tool

Open:

```text
http://localhost:3000
```

Expected output:

```text
Node.js API is running with MongoDB backend
```

---

# 13. Important Networking Concept

In this line:

```javascript
const MONGO_URI = "mongodb://mongodb:27017/companydb";
```

the hostname is `mongodb`.

### Why?
Because:
- Docker Compose creates an internal network
- service names become hostnames
- Node.js can reach MongoDB using service name

### Why not use localhost?
Because:
- `localhost` inside Node.js container means that same Node.js container
- MongoDB is running in another container

So correct host is:
```text
mongodb
```

not:
```text
localhost
```

---

# 14. Common Mistakes

## Mistake 1: Using `localhost` for MongoDB host
Wrong:
```javascript
mongodb://localhost:27017/companydb
```

### Why wrong?
Because Node.js and MongoDB are in separate containers.

Correct:
```javascript
mongodb://mongodb:27017/companydb
```

---

## Mistake 2: Forgetting `build` for custom Node.js app
If Node.js app uses custom code, using only `image` without building custom image may fail unless such image already exists.

---

## Mistake 3: Forgetting MongoDB volume
Without volume, database data may be lost on container recreation.

---

## Mistake 4: Not exposing Node.js port
Without:
```yaml
ports:
  - "3000:3000"
```
the API may not be reachable from host browser.

---

## Mistake 5: Thinking `depends_on` means full readiness
It controls startup order only, not guaranteed full MongoDB readiness.

For advanced deployment, healthchecks can be added.

---

# 15. Optional Improved Version with Environment Variables

A cleaner version uses environment variable for Mongo URI.

## Improved `server.js`

```javascript
const express = require("express");
const mongoose = require("mongoose");

const app = express();
const PORT = 3000;

const MONGO_URI = process.env.MONGO_URI || "mongodb://mongodb:27017/companydb";

mongoose.connect(MONGO_URI)
  .then(() => console.log("Connected to MongoDB successfully"))
  .catch((err) => console.log("MongoDB connection error:", err.message));

app.get("/", (req, res) => {
  res.send("Node.js API is running with MongoDB backend");
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

## Improved Compose snippet

```yaml
services:
  nodeapp:
    build: ./app
    ports:
      - "3000:3000"
    environment:
      - MONGO_URI=mongodb://mongodb:27017/companydb
    depends_on:
      - mongodb

  mongodb:
    image: mongo:6
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

### Why this is good?
Because:
- configuration is separated from code
- easier for development and deployment
- more reusable microservice design

---

# 16. Complete Final Files

## File 1: `app/package.json`

```json
{
  "name": "node-mongo-app",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.6.0"
  }
}
```

---

## File 2: `app/server.js`

```javascript
const express = require("express");
const mongoose = require("mongoose");

const app = express();
const PORT = 3000;

// MongoDB connection URI
const MONGO_URI = "mongodb://mongodb:27017/companydb";

// Connect to MongoDB
mongoose.connect(MONGO_URI)
  .then(() => console.log("Connected to MongoDB successfully"))
  .catch((err) => console.log("MongoDB connection error:", err.message));

app.get("/", (req, res) => {
  res.send("Node.js API is running with MongoDB backend");
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

---

## File 3: `app/Dockerfile`

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 3000
CMD ["npm", "start"]
```

---

## File 4: `docker-compose.yml`

```yaml
services:
  nodeapp:
    build: ./app
    container_name: node_api_app
    ports:
      - "3000:3000"
    depends_on:
      - mongodb

  mongodb:
    image: mongo:6
    container_name: mongodb_service
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

---

# 17. Viva / Exam Explanation

A student can explain the solution like this:

> This is a microservices deployment using Docker Compose where the Node.js application runs in one container and MongoDB runs in another container. The Node.js service uses the `build` field because it is a custom application created from local source code and Dockerfile. The MongoDB service uses the `image` field because it uses an official ready-made image. The Node.js application connects to MongoDB using the service name `mongodb`, which acts as hostname inside the Docker Compose network. A volume is used to persist MongoDB data.

---

# 18. Short Answer Version

> In this scenario, Docker Compose is used to deploy a Node.js API service and a MongoDB database as separate containers. The Node.js service uses the `build` field because it is built from local application source code, while the MongoDB service uses the `image` field because it is an official prebuilt image. Both containers communicate through the Docker Compose network, and MongoDB data is persisted using a named volume.

---

# 19. Final Conclusion

This scenario teaches a very important real-world deployment pattern:

- custom application containers are usually built using `build`
- standard infrastructure services usually use `image`
- multiple containers work together using Docker Compose
- service names become internal hostnames
- volumes preserve database data

So the correct solution is a Compose-based deployment containing:
- a custom-built Node.js API container
- an official MongoDB container
- proper networking
- proper persistence
- clear understanding of `build` vs `image`

