# FULL DETAILED SOLUTION: Deploying Java Spring Boot + PostgreSQL

---

## 1. Understanding the Scenario

### Real-Life Situation
A fintech startup runs a **Spring Boot backend service** that performs business logic, processes client requests, and manages financial application workflows.

This backend service needs a database to:
- store user records
- save transaction-related information
- maintain account or business data
- retrieve data for reporting and processing

The chosen database is **PostgreSQL**.

So the complete application needs two major services:

- **Spring Boot application service**
- **PostgreSQL database service**

This is a **multi-container deployment** because the complete system is made of more than one container, and both containers work together as one application.

---

## 2. Task Explanation

Students must:

- deploy a Spring Boot backend container
- deploy a PostgreSQL database container
- connect the backend to PostgreSQL properly
- understand how both services communicate
- use Docker Compose to orchestrate the full setup
- explain the configuration and files line by line
- run the application successfully

---

## 3. Concepts Covered

This scenario covers:

- multi-container deployment
- backend + database architecture
- Docker Compose orchestration
- service-to-service communication
- environment variable-based configuration
- persistent storage using volumes
- startup dependency management

---

## 4. Core Concept: Why Two Containers?

A backend application and a database are usually separated because they perform different roles.

### Spring Boot container
- runs the backend service
- handles API requests
- processes business rules
- interacts with database

### PostgreSQL container
- stores relational data
- handles SQL queries
- maintains persistent backend storage

### Why separate them?
Because:
- better modularity
- easier maintenance
- cleaner architecture
- easier scaling
- real-world DevOps deployment style
- better fault isolation

---

## 5. High-Level Architecture

```text
Client / Browser / API Tool
            ↓
   Spring Boot Backend Service
            ↓
      PostgreSQL Database
```

### Explanation
- client sends request to Spring Boot service
- Spring Boot processes the request
- if data is needed, Spring Boot talks to PostgreSQL
- PostgreSQL returns data
- Spring Boot sends final response back to the client

---

## 6. Project Structure

A simple project structure may look like this:

```text
springboot-postgres-demo/
│
├── docker-compose.yml
└── app/
    ├── Dockerfile
    └── springboot-app.jar
```

### Explanation
- `docker-compose.yml` defines both services
- `app/` contains the Spring Boot application jar and Dockerfile
- PostgreSQL uses an official ready-made image, so no custom Dockerfile is required for the database

> In a real project, the `.jar` file is produced after building the Spring Boot application using Maven or Gradle.

---

# 7. Full Implementation

We will assume that:
- the Spring Boot project has already been built
- the output jar file is available as `springboot-app.jar`
- Docker Compose will run that jar inside a Java container

This makes the example simple and deployment-focused.

---

## 7.1 `app/Dockerfile`

```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY springboot-app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## Line-by-line explanation of `Dockerfile`

### Line 1
```dockerfile
FROM eclipse-temurin:17-jre
```

### Explanation
This sets the base image to Java 17 runtime.

### Reasoning
Spring Boot applications run on Java.  
Since we are deploying a compiled jar file, we only need the **JRE** (Java Runtime Environment), not the full JDK.

`eclipse-temurin:17-jre` is a commonly used and stable Java runtime image.

---

### Line 2
```dockerfile
WORKDIR /app
```

### Explanation
This sets the working directory inside the container to `/app`.

### Reasoning
All later commands will run relative to this directory.  
It keeps files organized inside the container.

---

### Line 3
```dockerfile
COPY springboot-app.jar app.jar
```

### Explanation
This copies the local jar file into the container and renames it as `app.jar`.

### Reasoning
The Spring Boot application must be present inside the container so it can run.  
Renaming it to `app.jar` is optional, but it keeps the container file name simple.

---

### Line 4
```dockerfile
EXPOSE 8080
```

### Explanation
This documents that the application inside the container listens on port 8080.

### Reasoning
Spring Boot applications commonly run on port 8080 by default.

---

### Line 5
```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Explanation
This defines the command that starts the Spring Boot application when the container runs.

### Reasoning
The jar file is executed using Java.  
This makes the container start the backend service automatically.

---

# 8. Docker Compose Solution

## 8.1 `docker-compose.yml`

```yaml
services:
  postgres:
    image: postgres:15
    container_name: fintech_postgres
    restart: always
    environment:
      POSTGRES_DB: fintechdb
      POSTGRES_USER: fintechuser
      POSTGRES_PASSWORD: fintechpass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  springboot:
    build: ./app
    container_name: fintech_springboot
    restart: always
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/fintechdb
      SPRING_DATASOURCE_USERNAME: fintechuser
      SPRING_DATASOURCE_PASSWORD: fintechpass
    depends_on:
      - postgres

volumes:
  postgres_data:
```

---

# 9. Line-by-Line Explanation of `docker-compose.yml`

### Line 1
```yaml
services:
```

### Explanation
This starts the services section.

### Reasoning
Docker Compose manages a multi-container application as a set of services.

---

### Line 2
```yaml
postgres:
```

### Explanation
This defines the PostgreSQL service.

### Reasoning
The backend requires a relational database, so PostgreSQL is deployed as a separate service.

---

### Line 3
```yaml
image: postgres:15
```

### Explanation
This tells Docker to use the official PostgreSQL version 15 image.

### Reasoning
PostgreSQL already has a ready-made official image, so there is no need to build a custom database image for this example.

---

### Line 4
```yaml
container_name: fintech_postgres
```

### Explanation
This sets a custom name for the PostgreSQL container.

### Reasoning
A custom name makes the container easy to identify in Docker commands and logs.

---

### Line 5
```yaml
restart: always
```

### Explanation
This tells Docker to automatically restart the container if it stops.

### Reasoning
This improves service reliability.

---

### Line 6
```yaml
environment:
```

### Explanation
This starts the PostgreSQL environment variable section.

### Reasoning
The PostgreSQL image uses environment variables for initial database setup.

---

### Line 7
```yaml
POSTGRES_DB: fintechdb
```

### Explanation
This creates a database named `fintechdb`.

### Reasoning
The Spring Boot application needs a target database to store its application data.

---

### Line 8
```yaml
POSTGRES_USER: fintechuser
```

### Explanation
This creates a PostgreSQL user named `fintechuser`.

### Reasoning
It is better to use a dedicated application user rather than using the default superuser for normal application work.

---

### Line 9
```yaml
POSTGRES_PASSWORD: fintechpass
```

### Explanation
This sets the password for the `fintechuser` account.

### Reasoning
The backend will use this password to connect to PostgreSQL.

---

### Line 10
```yaml
ports:
```

### Explanation
This starts the PostgreSQL port mapping section.

### Reasoning
Port mapping allows optional direct access to PostgreSQL from the host system if needed.

---

### Line 11
```yaml
- "5432:5432"
```

### Explanation
This maps host port 5432 to container port 5432.

### Reasoning
5432 is the default PostgreSQL port.

---

### Line 12
```yaml
volumes:
```

### Explanation
This starts the volume mapping section.

### Reasoning
Database data should remain persistent even if the container is deleted or recreated.

---

### Line 13
```yaml
- postgres_data:/var/lib/postgresql/data
```

### Explanation
This mounts the named volume `postgres_data` into PostgreSQL's internal data directory.

### Reasoning
PostgreSQL stores its data in `/var/lib/postgresql/data`.  
Mounting a volume here ensures persistence of database data.

---

### Line 14
```yaml
springboot:
```

### Explanation
This defines the Spring Boot service.

### Reasoning
This is the main backend application service.

---

### Line 15
```yaml
build: ./app
```

### Explanation
This tells Docker Compose to build the Spring Boot image using the Dockerfile inside the `./app` folder.

### Reasoning
The Spring Boot service is a custom application, so Docker must build an image from local application files.

---

### Line 16
```yaml
container_name: fintech_springboot
```

### Explanation
This sets a custom name for the Spring Boot container.

### Reasoning
It makes the application container easier to identify during debugging and management.

---

### Line 17
```yaml
restart: always
```

### Explanation
This tells Docker to restart the Spring Boot container automatically if it stops.

### Reasoning
Improves application availability.

---

### Line 18
```yaml
ports:
```

### Explanation
This starts the Spring Boot port mapping section.

### Reasoning
The backend must be reachable from the host machine or API client.

---

### Line 19
```yaml
- "8080:8080"
```

### Explanation
This maps host port 8080 to container port 8080.

### Reasoning
Allows users or testers to access the backend at:

```text
http://localhost:8080
```

---

### Line 20
```yaml
environment:
```

### Explanation
This starts the environment variable section for Spring Boot.

### Reasoning
Spring Boot needs database connection configuration.

---

### Line 21
```yaml
SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/fintechdb
```

### Explanation
This sets the JDBC connection URL for PostgreSQL.

### Reasoning
- `jdbc:postgresql://` is the JDBC format for PostgreSQL
- `postgres` is the service name of the database container
- `5432` is the PostgreSQL port
- `fintechdb` is the database name

Docker Compose networking allows the backend container to reach the database container using the service name `postgres`.

---

### Line 22
```yaml
SPRING_DATASOURCE_USERNAME: fintechuser
```

### Explanation
This sets the database username used by Spring Boot.

### Reasoning
Must match the PostgreSQL user defined earlier.

---

### Line 23
```yaml
SPRING_DATASOURCE_PASSWORD: fintechpass
```

### Explanation
This sets the database password used by Spring Boot.

### Reasoning
Must match the PostgreSQL password configured in the database service.

---

### Line 24
```yaml
depends_on:
```

### Explanation
This starts the dependency section for Spring Boot.

### Reasoning
Spring Boot depends on PostgreSQL, so Compose should start the database service first.

---

### Line 25
```yaml
- postgres
```

### Explanation
This says the Spring Boot service depends on PostgreSQL.

### Reasoning
Ensures Docker Compose starts PostgreSQL before the backend.

---

### Line 26
```yaml
volumes:
```

### Explanation
This starts the top-level named volume definitions.

### Reasoning
Named volumes must be declared globally so Docker Compose can create and manage them.

---

### Line 27
```yaml
postgres_data:
```

### Explanation
This defines the named volume for PostgreSQL storage.

### Reasoning
Persistent database storage needs its own named volume.

---

# 10. Full Working Logic

When Docker Compose runs:

1. PostgreSQL image is pulled
2. Spring Boot image is built from local Dockerfile
3. PostgreSQL container starts
4. Database `fintechdb` and user `fintechuser` are created
5. Spring Boot container starts
6. Spring Boot reads datasource environment variables
7. Spring Boot tries to connect to PostgreSQL
8. Backend becomes available on port 8080
9. Client accesses the backend

---

# 11. Execution Steps

## Step 1: Create folder structure

```text
springboot-postgres-demo/
│
├── docker-compose.yml
└── app/
    ├── Dockerfile
    └── springboot-app.jar
```

---

## Step 2: Place the Spring Boot jar file
Copy the generated jar into:

```text
app/springboot-app.jar
```

---

## Step 3: Write Dockerfile
Use the Dockerfile shown earlier.

---

## Step 4: Write `docker-compose.yml`
Use the Compose file shown earlier.

---

## Step 5: Start the application

```bash
docker compose up --build
```

### Explanation
- `up` creates and starts the services
- `--build` ensures the Spring Boot image is built from the Dockerfile

---

## Step 6: Check running containers

```bash
docker ps
```

### Expected output
You should see:
- one PostgreSQL container
- one Spring Boot container

---

## Step 7: Access the backend

Open in browser or API client:

```text
http://localhost:8080
```

### Expected result
The exact result depends on the Spring Boot application routes.  
If a root endpoint exists, it should respond successfully.

---

# 12. Important Networking Concept

Students must understand this line carefully:

```yaml
SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/fintechdb
```

### Why use `postgres` as host?
Because:
- Docker Compose creates an internal network
- service names become hostnames
- the Spring Boot container reaches the PostgreSQL container using the service name `postgres`

### Why not use `localhost`?
Because:
- `localhost` inside the Spring Boot container means the Spring Boot container itself
- PostgreSQL is running in a different container

So correct host:
```text
postgres
```

not:
```text
localhost
```

---

# 13. Persistent Storage Explanation

Without volumes:
- database data may be lost when the container is removed
- application state dependent on DB may disappear

That is why we use:

```yaml
- postgres_data:/var/lib/postgresql/data
```

### Reasoning
Volumes keep the PostgreSQL database outside the temporary writable layer of the container.  
This preserves the data across container recreation.

---

# 14. Common Mistakes

## Mistake 1: Using `localhost` in datasource URL
Wrong:
```yaml
SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/fintechdb
```

### Why wrong?
Because Spring Boot and PostgreSQL are in separate containers.

Correct:
```yaml
SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/fintechdb
```

---

## Mistake 2: Username mismatch
Wrong example:
```yaml
POSTGRES_USER: fintechuser
SPRING_DATASOURCE_USERNAME: admin
```

### Why wrong?
Backend credentials must match the database credentials.

---

## Mistake 3: Password mismatch
If `POSTGRES_PASSWORD` and `SPRING_DATASOURCE_PASSWORD` differ, connection fails.

---

## Mistake 4: Forgetting volume
Without volume, PostgreSQL data may be lost.

---

## Mistake 5: Forgetting to expose Spring Boot port
Without:
```yaml
ports:
  - "8080:8080"
```
the backend may not be reachable from the host system.

---

## Mistake 6: Thinking `depends_on` guarantees full database readiness
`depends_on` controls startup order, but it does not always guarantee PostgreSQL is fully ready for connections at that exact moment.

For simple lab deployment, it is often acceptable.  
In advanced deployment, healthchecks can be added.

---

# 15. Optional Improved Version with Healthcheck

A stronger deployment can include PostgreSQL healthcheck.

```yaml
services:
  postgres:
    image: postgres:15
    container_name: fintech_postgres
    restart: always
    environment:
      POSTGRES_DB: fintechdb
      POSTGRES_USER: fintechuser
      POSTGRES_PASSWORD: fintechpass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U fintechuser -d fintechdb"]
      interval: 5s
      timeout: 3s
      retries: 10

  springboot:
    build: ./app
    container_name: fintech_springboot
    restart: always
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/fintechdb
      SPRING_DATASOURCE_USERNAME: fintechuser
      SPRING_DATASOURCE_PASSWORD: fintechpass
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  postgres_data:
```

### Why this is better?
Because:
- PostgreSQL health is checked
- Spring Boot starts only after PostgreSQL is healthy
- deployment becomes more reliable

---

# 16. Alternative Note for Real Spring Boot Projects

In real Spring Boot projects, the datasource is often configured inside:
- `application.properties`
- `application.yml`

But in containerized deployment, environment variables are commonly preferred because:
- easier deployment customization
- no hardcoding
- better portability across environments
- more DevOps-friendly approach

For example, Spring Boot can read:
- `SPRING_DATASOURCE_URL`
- `SPRING_DATASOURCE_USERNAME`
- `SPRING_DATASOURCE_PASSWORD`

directly from container environment variables.

---

# 17. Complete Final Files

## File 1: `app/Dockerfile`

```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY springboot-app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## File 2: `docker-compose.yml`

```yaml
services:
  postgres:
    image: postgres:15
    container_name: fintech_postgres
    restart: always
    environment:
      POSTGRES_DB: fintechdb
      POSTGRES_USER: fintechuser
      POSTGRES_PASSWORD: fintechpass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  springboot:
    build: ./app
    container_name: fintech_springboot
    restart: always
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/fintechdb
      SPRING_DATASOURCE_USERNAME: fintechuser
      SPRING_DATASOURCE_PASSWORD: fintechpass
    depends_on:
      - postgres

volumes:
  postgres_data:
```

---

# 18. Viva / Exam Explanation

A student can explain the solution like this:

> This is a multi-container backend + database deployment using Docker Compose. The Spring Boot application runs in one container, and PostgreSQL runs in another container. PostgreSQL is configured using environment variables to create the database, user, and password. The Spring Boot container is built from a custom Dockerfile that runs the jar file. It connects to PostgreSQL using the service name `postgres` inside the Docker Compose network. A named volume is used to persist PostgreSQL data. The full setup is started using `docker compose up --build`.

---

# 19. Short Answer Version

> In this scenario, Docker Compose is used to deploy a Spring Boot backend and a PostgreSQL database as separate containers. The PostgreSQL service uses an official image, while the Spring Boot service is built from a custom Dockerfile containing the application jar. The backend connects to PostgreSQL using the service name `postgres` through Docker Compose networking. Environment variables configure the database connection, and a named volume is used to persist database data.

---

# 20. Final Conclusion

This scenario teaches a very important backend deployment pattern:

- backend and database run in separate containers
- Docker Compose orchestrates both services
- service names act as internal hostnames
- environment variables configure datasource details
- volumes preserve database data
- the full deployment becomes modular, repeatable, and easy to manage

So the correct solution is a Docker Compose-based deployment containing:
- a Spring Boot backend container
- a PostgreSQL database container
- proper networking
- proper environment variables
- proper persistence
- correct backend + database architecture

