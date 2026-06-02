# Question 3: Environment Configuration for Microservices  
## Detailed Step-by-Step Solution with Full Implementation and Line-by-Line Explanation

---

# Real-Life Situation

A **Node.js backend service** must connect to **MongoDB**.  
Instead of hardcoding the database details directly into the source code, we will pass them using **environment variables**.

This is a very common real-world practice in microservices because:

- the same application may run in different environments
- database settings may change
- code becomes cleaner
- configuration becomes easier to manage

---

# Task

Create a **Docker Compose configuration** using **environment variables** to configure the database connection between:

- a **Node.js backend service**
- a **MongoDB database service**

---

# Concepts Covered

- Environment variables  
- Microservice configuration  
- Docker Compose  
- Service-to-service communication  
- Backend to database connectivity  

---

# 1. First Understand the Problem in Very Simple Words

Suppose you have two containers:

1. **Backend container** → runs your Node.js application  
2. **MongoDB container** → runs your database  

Now the backend must know:

- where MongoDB is running
- which port MongoDB is using
- which database name it should connect to

Instead of writing those values directly in code like this:

```js
const url = "mongodb://mongodb:27017/myappdb";
```

we will make the backend read them from **environment variables** like this:

```js
const DB_HOST = process.env.DB_HOST;
const DB_PORT = process.env.DB_PORT;
const DB_NAME = process.env.DB_NAME;
```

This makes the application more flexible.

---

# 2. What Are Environment Variables?

Environment variables are simple **name=value** pairs.

Example:

```env
DB_HOST=mongodb
DB_PORT=27017
DB_NAME=myappdb
```

These values are supplied to the container when it starts.

So instead of the backend application guessing where the database is, we explicitly tell it using variables.

---

# 3. Why Use Environment Variables?

Environment variables are useful because:

- they keep configuration outside code
- they make code reusable
- they make deployment easier
- they reduce hardcoding
- they help when moving from development to production

For example:

In local development:
```env
DB_HOST=mongodb
```

In cloud deployment:
```env
DB_HOST=prod-db.company.internal
```

Same code, different configuration.

---

# 4. What We Will Build

We will build the following:

- one **Node.js backend service**
- one **MongoDB service**
- one **Dockerfile** for backend
- one **docker-compose.yml** file
- one optional **.env** file
- one backend program that reads values from environment variables

---

# 5. Project Folder Structure

Create a folder like this:

```text
environment-config-microservices/
│
├── docker-compose.yml
├── .env
└── backend/
    ├── Dockerfile
    ├── package.json
    └── server.js
```

---

# 6. Step 1: Create `package.json`

Create a file named:

```text
backend/package.json
```

and add the following content:

```json
{
  "name": "backend-app",
  "version": "1.0.0",
  "description": "Node.js backend with MongoDB using environment variables",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^8.3.2"
  }
}
```

---

## Line-by-Line Explanation of `package.json`

```json
{
```
This starts the JSON object.

```json
  "name": "backend-app",
```
This is the name of the application.

```json
  "version": "1.0.0",
```
This is the project version.

```json
  "description": "Node.js backend with MongoDB using environment variables",
```
This describes what the application does.

```json
  "main": "server.js",
```
This tells Node.js that the main file is `server.js`.

```json
  "scripts": {
    "start": "node server.js"
  },
```
This defines a script named `start`.  
When we run:

```bash
npm start
```

Node.js will execute:

```bash
node server.js
```

```json
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^8.3.2"
  }
```
These are required libraries:

- `express` → used to create the backend server
- `mongoose` → used to connect Node.js with MongoDB

```json
}
```
This ends the JSON object.

---

# 7. Step 2: Create the Backend Application

Create a file named:

```text
backend/server.js
```

and write the following code:

```javascript
const express = require("express");
const mongoose = require("mongoose");

const app = express();
const PORT = process.env.PORT || 5000;

const DB_HOST = process.env.DB_HOST || "mongodb";
const DB_PORT = process.env.DB_PORT || "27017";
const DB_NAME = process.env.DB_NAME || "myappdb";

const mongoURL = `mongodb://${DB_HOST}:${DB_PORT}/${DB_NAME}`;

mongoose
  .connect(mongoURL)
  .then(() => {
    console.log("Connected to MongoDB successfully");
  })
  .catch((error) => {
    console.log("MongoDB connection error:", error);
  });

app.get("/", (req, res) => {
  res.send("Node.js backend is running and connected to MongoDB using environment variables");
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

---

## Line-by-Line Explanation of `server.js`

```javascript
const express = require("express");
```
This imports the **Express** framework.  
Express helps us create a web server easily.

```javascript
const mongoose = require("mongoose");
```
This imports **Mongoose**, which helps Node.js connect to MongoDB.

```javascript
const app = express();
```
This creates the Express application object.

```javascript
const PORT = process.env.PORT || 5000;
```
This reads the `PORT` value from environment variables.  
If no environment variable is provided, it uses `5000`.

```javascript
const DB_HOST = process.env.DB_HOST || "mongodb";
```
This reads the database host from environment variable `DB_HOST`.  
If not present, it uses `mongodb`.

```javascript
const DB_PORT = process.env.DB_PORT || "27017";
```
This reads the MongoDB port from environment variable `DB_PORT`.  
Default value is `27017`.

```javascript
const DB_NAME = process.env.DB_NAME || "myappdb";
```
This reads the database name from environment variable `DB_NAME`.  
If nothing is passed, it uses `myappdb`.

```javascript
const mongoURL = `mongodb://${DB_HOST}:${DB_PORT}/${DB_NAME}`;
```
This creates the MongoDB connection string dynamically.

If the values are:

- `DB_HOST=mongodb`
- `DB_PORT=27017`
- `DB_NAME=myappdb`

then the final URL becomes:

```text
mongodb://mongodb:27017/myappdb
```

```javascript
mongoose
  .connect(mongoURL)
```
This tells Mongoose to connect to MongoDB using that URL.

```javascript
  .then(() => {
    console.log("Connected to MongoDB successfully");
  })
```
If the connection is successful, it prints a success message.

```javascript
  .catch((error) => {
    console.log("MongoDB connection error:", error);
  });
```
If the connection fails, it prints the error.

```javascript
app.get("/", (req, res) => {
  res.send("Node.js backend is running and connected to MongoDB using environment variables");
});
```
This creates a route for `/`.  
When you visit the backend in a browser, this message will appear.

```javascript
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```
This starts the backend server on the selected port.

---

# 8. Step 3: Create the Dockerfile

Create a file named:

```text
backend/Dockerfile
```

and add the following:

```dockerfile
FROM node:18

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm", "start"]
```

---

## Line-by-Line Explanation of `Dockerfile`

```dockerfile
FROM node:18
```
This tells Docker to use the official Node.js version 18 image as the base image.

```dockerfile
WORKDIR /app
```
This sets the working directory inside the container to `/app`.

Any command after this runs inside `/app`.

```dockerfile
COPY package.json .
```
This copies `package.json` from your local backend folder into the container.

```dockerfile
RUN npm install
```
This installs all dependencies listed in `package.json`.

```dockerfile
COPY . .
```
This copies all remaining files from the backend folder into the container.

```dockerfile
EXPOSE 5000
```
This tells Docker that the application inside the container uses port `5000`.

```dockerfile
CMD ["npm", "start"]
```
This is the command that runs when the container starts.

It executes:

```bash
npm start
```

which runs the Node.js server.

---

# 9. Step 4: Create the `.env` File

Create a file named:

```text
.env
```

and write:

```env
PORT=5000
DB_HOST=mongodb
DB_PORT=27017
DB_NAME=myappdb
MONGO_INITDB_DATABASE=myappdb
```

---

## Line-by-Line Explanation of `.env`

```env
PORT=5000
```
This is the port on which the backend app will run.

```env
DB_HOST=mongodb
```
This tells the backend that the database host is `mongodb`.

Important point:  
`mongodb` is the **service name** in Docker Compose, not an IP address.

```env
DB_PORT=27017
```
This tells the backend that MongoDB is listening on port `27017`.

```env
DB_NAME=myappdb
```
This is the database name to connect to.

```env
MONGO_INITDB_DATABASE=myappdb
```
This tells the MongoDB container to initialize a database with this name.

---

# 10. Step 5: Create the `docker-compose.yml` File

Create a file named:

```text
docker-compose.yml
```

and write:

```yaml
version: "3.8"

services:
  backend:
    build: ./backend
    container_name: node_backend
    ports:
      - "${PORT}:5000"
    environment:
      - PORT=${PORT}
      - DB_HOST=${DB_HOST}
      - DB_PORT=${DB_PORT}
      - DB_NAME=${DB_NAME}
    depends_on:
      - mongodb

  mongodb:
    image: mongo:6
    container_name: mongodb_container
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_DATABASE=${MONGO_INITDB_DATABASE}
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

---

# 11. Line-by-Line Explanation of `docker-compose.yml`

## First line

```yaml
version: "3.8"
```
This defines the version of Docker Compose file syntax being used.

---

## Start of services

```yaml
services:
```
This starts the list of services that Docker Compose will create.

Each service usually becomes one container.

---

## Backend service

```yaml
  backend:
```
This creates a service named `backend`.

This service will run the Node.js application.

---

```yaml
    build: ./backend
```
This tells Docker Compose to build the backend image from the `backend` folder.

Docker will look for a `Dockerfile` inside that folder.

---

```yaml
    container_name: node_backend
```
This gives a custom name to the backend container.

Without this, Docker would assign an automatic name.

---

```yaml
    ports:
      - "${PORT}:5000"
```
This maps a port from your computer to the container.

- left side = host machine port
- right side = container port

If `PORT=5000`, then this becomes:

```yaml
- "5000:5000"
```

Meaning:
- open browser on `localhost:5000`
- request reaches backend container port `5000`

---

```yaml
    environment:
```
This starts the environment variables section for the backend container.

---

```yaml
      - PORT=${PORT}
```
This passes the value of `PORT` from the `.env` file into the backend container.

---

```yaml
      - DB_HOST=${DB_HOST}
```
This passes the value of `DB_HOST` to the backend container.

The backend will read this using:

```js
process.env.DB_HOST
```

---

```yaml
      - DB_PORT=${DB_PORT}
```
This passes the database port to the backend container.

---

```yaml
      - DB_NAME=${DB_NAME}
```
This passes the database name to the backend container.

---

```yaml
    depends_on:
      - mongodb
```
This tells Docker Compose to start the `mongodb` service before starting `backend`.

Important note:  
This controls startup order, but does not guarantee MongoDB is fully ready before backend tries to connect.

Still, for beginner-level setup, this is acceptable.

---

## MongoDB service

```yaml
  mongodb:
```
This creates a service named `mongodb`.

This name is very important because the backend uses:

```env
DB_HOST=mongodb
```

So the backend can find the database using that service name.

---

```yaml
    image: mongo:6
```
This tells Docker Compose to use the official MongoDB version 6 image.

---

```yaml
    container_name: mongodb_container
```
This gives a custom name to the MongoDB container.

---

```yaml
    ports:
      - "27017:27017"
```
This maps MongoDB port 27017 from the container to your computer.

This is useful if you want to connect to MongoDB from your host machine too.

---

```yaml
    environment:
```
This starts the environment variables section for MongoDB.

---

```yaml
      - MONGO_INITDB_DATABASE=${MONGO_INITDB_DATABASE}
```
This tells MongoDB to initialize a database with the name stored in `.env`.

---

```yaml
    volumes:
      - mongo_data:/data/db
```
This attaches a named volume called `mongo_data` to MongoDB’s internal data folder.

Why this is needed:
- if the container stops
- or gets deleted and recreated

your database data will still remain stored in the volume.

---

## Volumes section

```yaml
volumes:
  mongo_data:
```
This defines the named volume `mongo_data`.

Docker manages this volume automatically.

---

# 12. Very Important Beginner Concept: Why `DB_HOST=mongodb` and Not `localhost`?

This confuses many beginners.

Inside the backend container:

- `localhost` means **the backend container itself**
- it does **not** mean the MongoDB container

So this is wrong:

```env
DB_HOST=localhost
```

because the backend would then try to find MongoDB inside its own container.

Correct:

```env
DB_HOST=mongodb
```

because `mongodb` is the Docker Compose service name.

Docker Compose automatically creates a network and allows services to communicate using service names.

---

# 13. How the Connection Works Internally

Docker Compose creates a private network for all services.

So:

- backend service can reach mongodb service
- mongodb service can reach backend service if needed

When backend builds the URL:

```text
mongodb://mongodb:27017/myappdb
```

it means:

- protocol = mongodb
- host = mongodb service
- port = 27017
- database = myappdb

---

# 14. Commands to Run the Project

Open terminal in the main project folder and run:

## Build and start everything

```bash
docker-compose up --build
```

### What this does
- reads `docker-compose.yml`
- reads values from `.env`
- builds the backend image
- pulls MongoDB image
- creates containers
- creates network
- creates volume
- starts services

---

## Run in detached mode

```bash
docker-compose up --build -d
```

This runs the containers in the background.

---

## See running containers

```bash
docker ps
```

This shows the active containers.

---

## See logs of all services

```bash
docker-compose logs
```

---

## See logs of backend only

```bash
docker-compose logs backend
```

---

## Stop the project

```bash
docker-compose down
```

This stops and removes the containers.

---

## Stop and remove volume also

```bash
docker-compose down -v
```

This also deletes stored MongoDB data.

---

# 15. Expected Output

When everything runs correctly, the logs may show:

```text
Connected to MongoDB successfully
Server is running on port 5000
```

Now open the browser and visit:

```text
http://localhost:5000
```

Expected response:

```text
Node.js backend is running and connected to MongoDB using environment variables
```

---

# 16. Full Implementation Together

---

## `backend/package.json`

```json
{
  "name": "backend-app",
  "version": "1.0.0",
  "description": "Node.js backend with MongoDB using environment variables",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^8.3.2"
  }
}
```

---

## `backend/server.js`

```javascript
const express = require("express");
const mongoose = require("mongoose");

const app = express();
const PORT = process.env.PORT || 5000;

const DB_HOST = process.env.DB_HOST || "mongodb";
const DB_PORT = process.env.DB_PORT || "27017";
const DB_NAME = process.env.DB_NAME || "myappdb";

const mongoURL = `mongodb://${DB_HOST}:${DB_PORT}/${DB_NAME}`;

mongoose
  .connect(mongoURL)
  .then(() => {
    console.log("Connected to MongoDB successfully");
  })
  .catch((error) => {
    console.log("MongoDB connection error:", error);
  });

app.get("/", (req, res) => {
  res.send("Node.js backend is running and connected to MongoDB using environment variables");
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

---

## `backend/Dockerfile`

```dockerfile
FROM node:18

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm", "start"]
```

---

## `.env`

```env
PORT=5000
DB_HOST=mongodb
DB_PORT=27017
DB_NAME=myappdb
MONGO_INITDB_DATABASE=myappdb
```

---

## `docker-compose.yml`

```yaml
version: "3.8"

services:
  backend:
    build: ./backend
    container_name: node_backend
    ports:
      - "${PORT}:5000"
    environment:
      - PORT=${PORT}
      - DB_HOST=${DB_HOST}
      - DB_PORT=${DB_PORT}
      - DB_NAME=${DB_NAME}
    depends_on:
      - mongodb

  mongodb:
    image: mongo:6
    container_name: mongodb_container
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_DATABASE=${MONGO_INITDB_DATABASE}
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

---

# 17. Simple Analogy for Naive Learners

Think of it like this:

- **MongoDB container** = database room  
- **Backend container** = application room  
- **Environment variables** = address details written on a slip  
- **Docker Compose** = building manager who creates both rooms and connects them  

Backend says:

- database room name = mongodb
- database port = 27017
- database name = myappdb

Then it connects correctly.

---

# 18. Common Mistakes Beginners Make

## Mistake 1: Using localhost
Wrong:

```env
DB_HOST=localhost
```

Correct:

```env
DB_HOST=mongodb
```

---

## Mistake 2: Forgetting dependencies
If `npm install` is not run, backend will fail.

---

## Mistake 3: Wrong build path
If your backend folder is `./backend`, then Compose must say:

```yaml
build: ./backend
```

---

## Mistake 4: Wrong port mapping
If app listens on 5000 inside container, right side must be 5000.

---

## Mistake 5: Expecting `depends_on` to guarantee DB readiness
`depends_on` only starts MongoDB first.  
It does not fully guarantee database is ready to accept connections immediately.

---

# 19. Short Exam Answer

A Docker Compose configuration can use environment variables to pass database connection details such as host, port, and database name to a Node.js backend service. These variables may be defined in a `.env` file and injected into the backend through the `environment` section. The backend then reads them using `process.env` and dynamically builds the MongoDB connection string. In Docker Compose, the backend can use the database service name, such as `mongodb`, as the host name for inter-container communication.

---

# 20. Final Conclusion

In this solution:

- we created a Node.js backend
- we created a MongoDB service
- we used environment variables for flexible configuration
- we used Docker Compose to connect both services
- we explained each line in simple beginner-friendly language

This is one of the most common and important patterns in containerized microservices.

---

# 21. Quick Run Checklist

Before running, ensure:

- Docker Desktop is installed
- project folder structure is correct
- `.env` file is present
- `Dockerfile` is inside `backend`
- `server.js` and `package.json` are correct

Then run:

```bash
docker-compose up --build
```

---

# End of Markdown File
