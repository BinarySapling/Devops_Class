# FULL DETAILED SOLUTION: Multi-Container Deployment (WordPress + MySQL)

---

## 1. Understanding the Scenario

### Real-Life Situation
A blogging platform needs two major components:

- **WordPress** for the website and content management
- **MySQL** for storing posts, users, settings, comments, and other website data

This is a classic **multi-container application** because the full system is not made of just one container.  
Instead, it is made of multiple services that work together.

### Why are two containers needed?
Because WordPress and MySQL do different jobs:

- **WordPress container**
  - serves the website
  - provides admin dashboard
  - manages themes, plugins, posts, pages, and users

- **MySQL container**
  - stores website data
  - handles database operations
  - keeps the site content persistent

So WordPress depends on MySQL.

---

## 2. Task Explanation

Students must:

- deploy WordPress using Docker Compose
- deploy MySQL using Docker Compose
- connect both services correctly
- understand how the services communicate
- run the full blogging platform using one Compose file
- explain the implementation line by line

---

## 3. Concepts Covered

This scenario covers:

- multi-container applications
- Docker Compose orchestration
- inter-service communication
- persistent storage using volumes
- environment variable-based configuration
- service dependency management

---

## 4. Core Concept: What is a Multi-Container Application?

A **multi-container application** is an application where different parts run in separate containers.

For example:

- frontend in one container
- backend in another container
- database in another container

In this scenario:

- WordPress runs in one container
- MySQL runs in another container

These containers work together as one complete application.

### Why use separate containers?
Because:
- each service has a dedicated responsibility
- easier maintenance
- independent updates
- better isolation
- real-world deployment style

---

## 5. High-Level Architecture

```text
User Browser
     ↓
 WordPress Container
     ↓
   MySQL Container
```

### Explanation
- user opens website in browser
- browser reaches WordPress
- WordPress fetches or stores data in MySQL
- MySQL returns data to WordPress
- WordPress shows final webpage to user

---

## 6. Startup Flow

The logical startup order is:

1. MySQL container starts
2. WordPress container starts
3. WordPress connects to MySQL
4. User accesses the site in browser

---

## 7. Project Structure

A simple Compose-based deployment may look like this:

```text
wordpress-mysql-demo/
│
└── docker-compose.yml
```

### Explanation
For this example, we only need one file:
- `docker-compose.yml`

Why?
Because:
- WordPress uses an official ready-made image
- MySQL also uses an official ready-made image
- Docker Compose can configure both services directly

So no custom Dockerfile is required in this basic example.

---

# 8. Full Docker Compose Solution

## 8.1 `docker-compose.yml`

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: wordpress_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    volumes:
      - mysql_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress_db
    depends_on:
      - mysql
    volumes:
      - wordpress_data:/var/www/html

volumes:
  mysql_data:
  wordpress_data:
```

---

# 9. Line-by-Line Explanation of the Compose File

---

## Line 1
```yaml
services:
```

### Explanation
This starts the `services` section.

### Reasoning
Docker Compose manages applications as a collection of services.  
Each service usually corresponds to one container.

---

## Line 2
```yaml
mysql:
```

### Explanation
This defines the MySQL service.

### Reasoning
The blogging platform needs a database, so MySQL is defined as a separate service.

---

## Line 3
```yaml
image: mysql:8.0
```

### Explanation
This tells Docker to use the official MySQL 8.0 image.

### Reasoning
Using an official image is easier, reliable, and standard for lab deployment.

---

## Line 4
```yaml
container_name: wordpress_mysql
```

### Explanation
This assigns a custom name to the MySQL container.

### Reasoning
It makes the container easier to identify during inspection, logs, and troubleshooting.

---

## Line 5
```yaml
restart: always
```

### Explanation
This tells Docker to always restart the container if it stops.

### Reasoning
This improves service availability.  
If the container crashes or the system restarts, Docker tries to bring it back automatically.

---

## Line 6
```yaml
environment:
```

### Explanation
This starts the environment variable section for MySQL.

### Reasoning
The MySQL image uses environment variables for initial setup and configuration.

---

## Line 7
```yaml
MYSQL_ROOT_PASSWORD: rootpassword
```

### Explanation
This sets the root password for MySQL.

### Reasoning
The root user is the main administrative user in MySQL.  
A password is required for secure database initialization.

---

## Line 8
```yaml
MYSQL_DATABASE: wordpress_db
```

### Explanation
This creates a database named `wordpress_db`.

### Reasoning
WordPress needs a dedicated database to store website content and settings.

---

## Line 9
```yaml
MYSQL_USER: wpuser
```

### Explanation
This creates a custom MySQL user named `wpuser`.

### Reasoning
It is better practice to use a specific application user instead of using the root user for daily application access.

---

## Line 10
```yaml
MYSQL_PASSWORD: wppassword
```

### Explanation
This sets the password for the `wpuser` account.

### Reasoning
WordPress will use this username and password to connect to MySQL.

---

## Line 11
```yaml
volumes:
```

### Explanation
This starts the volume mapping section for MySQL.

### Reasoning
Database data must persist even if the container is deleted or recreated.

---

## Line 12
```yaml
- mysql_data:/var/lib/mysql
```

### Explanation
This mounts the named volume `mysql_data` into MySQL's internal data directory.

### Reasoning
MySQL stores its data in `/var/lib/mysql`.  
Mounting a volume here ensures database content is not lost.

---

## Line 13
```yaml
wordpress:
```

### Explanation
This defines the WordPress service.

### Reasoning
This is the main blogging platform application.

---

## Line 14
```yaml
image: wordpress:latest
```

### Explanation
This tells Docker to use the official latest WordPress image.

### Reasoning
The official image already contains WordPress and a web server, so no custom setup is needed for a basic deployment.

---

## Line 15
```yaml
container_name: wordpress_app
```

### Explanation
This gives a custom name to the WordPress container.

### Reasoning
This makes it easier to identify in Docker commands and logs.

---

## Line 16
```yaml
restart: always
```

### Explanation
This ensures WordPress restarts automatically if it stops.

### Reasoning
This improves reliability and availability.

---

## Line 17
```yaml
ports:
```

### Explanation
This starts the port mapping section for WordPress.

### Reasoning
Port mapping is needed so that users can access WordPress from the host browser.

---

## Line 18
```yaml
- "8080:80"
```

### Explanation
This maps host port `8080` to container port `80`.

### Reasoning
Inside the container, WordPress serves web traffic on port 80.  
On the host machine, the site will be accessed through port 8080.

So users open:
```text
http://localhost:8080
```

---

## Line 19
```yaml
environment:
```

### Explanation
This starts the environment variable section for WordPress.

### Reasoning
WordPress needs database connection settings to connect to MySQL.

---

## Line 20
```yaml
WORDPRESS_DB_HOST: mysql:3306
```

### Explanation
This tells WordPress where the database is located.

### Reasoning
The database host is `mysql`, which is the service name.  
Docker Compose automatically creates networking between services, so WordPress can reach MySQL using the name `mysql`.

Port `3306` is the default MySQL port.

---

## Line 21
```yaml
WORDPRESS_DB_USER: wpuser
```

### Explanation
This sets the database username for WordPress.

### Reasoning
WordPress must log in to MySQL using the same user created in the MySQL configuration.

---

## Line 22
```yaml
WORDPRESS_DB_PASSWORD: wppassword
```

### Explanation
This sets the database password for WordPress.

### Reasoning
It must match `MYSQL_PASSWORD` defined in the MySQL service.

---

## Line 23
```yaml
WORDPRESS_DB_NAME: wordpress_db
```

### Explanation
This tells WordPress which database to use.

### Reasoning
It must match the database created by MySQL.

---

## Line 24
```yaml
depends_on:
```

### Explanation
This starts the dependency definition for WordPress.

### Reasoning
WordPress depends on MySQL, so Compose should start MySQL first.

---

## Line 25
```yaml
- mysql
```

### Explanation
This says WordPress depends on the `mysql` service.

### Reasoning
Compose starts MySQL before WordPress.

---

## Line 26
```yaml
volumes:
```

### Explanation
This starts the volume section for WordPress.

### Reasoning
WordPress content such as themes, plugins, uploads, and configuration should remain persistent.

---

## Line 27
```yaml
- wordpress_data:/var/www/html
```

### Explanation
This mounts the named volume `wordpress_data` into WordPress application directory.

### Reasoning
This keeps website files persistent even if the container is recreated.

---

## Line 28
```yaml
volumes:
```

### Explanation
This starts the top-level named volume definitions.

### Reasoning
Named volumes must be declared so Docker Compose can create and manage them.

---

## Line 29
```yaml
mysql_data:
```

### Explanation
This defines a named volume for MySQL data.

### Reasoning
Database storage needs persistent named volume support.

---

## Line 30
```yaml
wordpress_data:
```

### Explanation
This defines a named volume for WordPress data.

### Reasoning
Website content and configuration should remain persistent.

---

# 10. Why These Environment Variables Are Needed

WordPress cannot work unless it knows:

- database host
- database user
- database password
- database name

That is why these lines are required:

```yaml
WORDPRESS_DB_HOST: mysql:3306
WORDPRESS_DB_USER: wpuser
WORDPRESS_DB_PASSWORD: wppassword
WORDPRESS_DB_NAME: wordpress_db
```

Similarly, MySQL also needs initialization settings:

```yaml
MYSQL_ROOT_PASSWORD: rootpassword
MYSQL_DATABASE: wordpress_db
MYSQL_USER: wpuser
MYSQL_PASSWORD: wppassword
```

---

# 11. Full Working Logic

When Docker Compose runs:

1. MySQL image is pulled
2. WordPress image is pulled
3. MySQL container is created
4. MySQL database `wordpress_db` is initialized
5. MySQL user `wpuser` is created
6. WordPress container is created
7. WordPress reads DB connection settings
8. WordPress connects to MySQL
9. User opens the site in browser
10. WordPress installation page or website appears

---

# 12. Execution Steps

## Step 1: Create a project folder
Create a folder named:

```text
wordpress-mysql-demo
```

Inside it, create:

```text
docker-compose.yml
```

---

## Step 2: Write the Compose file
Copy the earlier full Compose configuration into `docker-compose.yml`.

---

## Step 3: Start the application
Run:

```bash
docker compose up -d
```

### Explanation
- `docker compose` uses the Compose file
- `up` creates and starts containers
- `-d` means detached mode, so containers run in background

---

## Step 4: Check running containers
Run:

```bash
docker ps
```

### Expected result
You should see:
- one WordPress container
- one MySQL container

---

## Step 5: Open in browser
Go to:

```text
http://localhost:8080
```

### Expected output
You should see the WordPress setup page or the WordPress site.

---

# 13. Commands Used in This Scenario

## Command 1
```bash
docker compose up -d
```

### Explanation
Starts the multi-container application in background.

### Reasoning
This is the main deployment command for Compose-based applications.

---

## Command 2
```bash
docker ps
```

### Explanation
Shows running containers.

### Reasoning
Used to verify that both WordPress and MySQL are active.

---

## Command 3
```bash
docker compose down
```

### Explanation
Stops and removes the Compose application containers.

### Reasoning
Useful for cleanup or restarting the full system.

---

# 14. Persistent Storage Explanation

Without volumes:
- database data may be lost
- uploaded media may disappear
- plugin/theme changes may vanish

That is why we use:

```yaml
- mysql_data:/var/lib/mysql
- wordpress_data:/var/www/html
```

### Reasoning
Volumes keep data outside the temporary writable layer of the container.

This means:
- container can be deleted
- data can still survive
- application state is preserved

---

# 15. Important Networking Concept

Students must understand this line carefully:

```yaml
WORDPRESS_DB_HOST: mysql:3306
```

### Why `mysql` and not `localhost`?
Because:
- `localhost` inside WordPress container means the WordPress container itself
- MySQL is running in a separate container
- Docker Compose creates a shared network
- services can communicate using service names

So:
- service name = hostname
- `mysql` becomes the internal DNS name of database service

This is a very important Docker Compose concept.

---

# 16. Common Mistakes

## Mistake 1: Using `localhost` for database host
Wrong:
```yaml
WORDPRESS_DB_HOST: localhost:3306
```

### Why wrong?
Because WordPress is inside its own container.  
`localhost` would point to that same container, not the MySQL container.

Correct:
```yaml
WORDPRESS_DB_HOST: mysql:3306
```

---

## Mistake 2: Password mismatch
Wrong example:
```yaml
MYSQL_PASSWORD: abc123
WORDPRESS_DB_PASSWORD: xyz456
```

### Why wrong?
WordPress must use the exact same credentials created in MySQL.

---

## Mistake 3: Forgetting volumes
Without volumes, data may disappear when containers are removed.

---

## Mistake 4: Not exposing WordPress port
If this is missing:
```yaml
ports:
  - "8080:80"
```
then browser may not access the site.

---

## Mistake 5: Assuming `depends_on` means full readiness
`depends_on` starts MySQL first, but does not always guarantee MySQL is fully ready at that exact moment.

For this lab case, it is acceptable.  
In advanced cases, healthchecks may also be used.

---

# 17. Improved Version with Healthcheck (Optional Advanced)

If you want a stronger practical version, you may use healthcheck.

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: wordpress_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-prootpassword"]
      interval: 5s
      timeout: 3s
      retries: 10

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress_db
    depends_on:
      mysql:
        condition: service_healthy
    volumes:
      - wordpress_data:/var/www/html

volumes:
  mysql_data:
  wordpress_data:
```

### Why is this better?
Because:
- MySQL is checked for health
- WordPress starts only after MySQL is healthy
- more reliable orchestration

---

# 18. Full Final Files

## File: `docker-compose.yml`

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: wordpress_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    volumes:
      - mysql_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress_db
    depends_on:
      - mysql
    volumes:
      - wordpress_data:/var/www/html

volumes:
  mysql_data:
  wordpress_data:
```

---

# 19. Viva / Exam Explanation

A student can explain the solution like this:

> WordPress and MySQL together form a multi-container application. WordPress is the application service and MySQL is the database service. Docker Compose is used to orchestrate both services in a single `docker-compose.yml` file. The MySQL container is configured using environment variables to create the database and user. The WordPress container uses corresponding environment variables to connect to MySQL through the service name `mysql`. Volumes are used to persist database data and WordPress files. The application is started using `docker compose up -d`.

---

# 20. Short Answer Version

> In this scenario, Docker Compose is used to deploy a multi-container blogging platform with WordPress and MySQL. The MySQL service stores the database, while the WordPress service provides the website interface. Both services are configured inside one Compose file and communicate through Docker Compose networking using the service name `mysql`. Volumes are used for persistence, and the application is started using `docker compose up -d`.

---

# 21. Final Conclusion

This scenario is an excellent example of **Docker Compose orchestration** for a **multi-container application**.

It teaches students that:
- one application may need multiple services
- containers can work together through Compose
- service names act as hostnames
- environment variables configure services
- volumes preserve application data
- Compose makes deployment simple and repeatable

So the correct full solution is a Docker Compose file that deploys:
- **WordPress**
- **MySQL**
- proper environment variables
- proper port mapping
- proper persistence
- proper dependency orchestration

