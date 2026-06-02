---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 10: Maven and Docker Integration"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Dockerizing Maven applications and pushing images to registries"
---

# Scenario 10  
## Maven and Docker Integration

**Syllabus Coverage:**  
Maven and Docker integration, Dockerizing Maven-based applications, dockerfile-maven-plugin, container image build, tagging, and pushing artifacts to registries.

---

# Question 10

A company wants every successful Maven build to automatically create a Docker image and push it to Docker Hub or GitHub Container Registry.

As a DevOps engineer:

1. Explain the benefits of integrating Maven with Docker in DevOps pipelines.  
2. Describe how Maven can package the application and create a Docker image.  
3. Show how `dockerfile-maven-plugin` or similar tooling can build and push the container image after packaging.  
4. Explain how the Docker image can be verified locally and pushed to a registry.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain why Maven and Docker are integrated.
- Create a Maven-based Java application.
- Package the application into a JAR.
- Write a Dockerfile for a Maven application.
- Configure Docker image build through Maven.
- Tag Docker images properly.
- Push Docker images to registries.
- Understand CI/CD relevance of containerized builds.

---

# Problem Understanding

The company wants an automated workflow:

```text
Code Commit
   ↓
Maven Build
   ↓
JAR Package
   ↓
Docker Image Build
   ↓
Registry Push
   ↓
Deployment Ready
```

The goal is to reduce manual work and make releases consistent.

---

# Why Maven and Docker Integration Is Useful

Maven handles:

- Compilation
- Dependency management
- Testing
- Packaging

Docker handles:

- Container image creation
- Runtime environment standardization
- Portable deployment
- Registry-based distribution

Together, they support modern DevOps automation.

---

# What Problem Does This Solve?

Without automation:

- Developers build JAR manually.
- Docker image is created manually.
- Tags may be inconsistent.
- Wrong artifact may enter image.
- Push to registry may be forgotten.
- Deployment may use outdated builds.

With Maven-Docker integration, the process becomes repeatable.

---

# Maven vs Docker Roles

| Tool | Responsibility |
|---|---|
| Maven | Builds Java artifact |
| Docker | Packages runtime environment |
| Registry | Stores and distributes images |
| CI/CD | Automates the full process |

Maven creates the JAR, Docker packages the JAR with Java runtime.

---

# Implementation Goal

We will implement a simple Maven application and Dockerize it.

The workflow will be:

```bash
mvn clean package
docker build -t username/maven-docker-app:1.0.0 .
docker run username/maven-docker-app:1.0.0
docker push username/maven-docker-app:1.0.0
```

Then we will automate image build through Maven plugin.

---

# Step 1: Create Project Folder

```bash
mkdir maven-docker-app
cd maven-docker-app
```

This is the root directory of the application.

It will contain:

```text
pom.xml
Dockerfile
src/
target/
```

---

# Step 2: Create Maven Directory Structure

For Linux, macOS, or Git Bash:

```bash
mkdir -p src/main/java/com/company/app
mkdir -p src/test/java/com/company/app
```

For Windows Command Prompt:

```cmd
mkdir src\main\java\com\company\app
mkdir src\test\java\com\company\app
```

---

# Step 3: Create Java Application

Create file:

```text
src/main/java/com/company/app/App.java
```

Add this code:

```java
package com.company.app;

public class App {
    public static void main(String[] args) {
        System.out.println("Maven Docker App is running inside container.");
    }

    public static String status() {
        return "CONTAINER_READY";
    }
}
```

---

# Explanation of App.java

```java
main()
```

This method runs when the container starts.

```java
System.out.println(...)
```

Prints a confirmation message.

```java
status()
```

Used for unit testing.

This confirms that the application logic works before packaging.

---

# Step 4: Create Unit Test

Create file:

```text
src/test/java/com/company/app/AppTest.java
```

Add this code:

```java
package com.company.app;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class AppTest {
    @Test
    public void testStatus() {
        assertEquals("CONTAINER_READY", App.status());
    }
}
```

---

# Why Unit Test Is Important

Before creating a Docker image, Maven should ensure that:

- Source code compiles.
- Unit tests pass.
- The packaged JAR is valid.

A Docker image should not be created from broken code.

---

# Step 5: Create pom.xml

Create:

```text
pom.xml
```

The POM will include:

- Project metadata
- Java version
- JUnit dependency
- Surefire plugin
- JAR plugin
- Docker plugin configuration

---

# Basic POM Part 1

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company</groupId>
    <artifactId>maven-docker-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
```

---

# Basic POM Part 2

```xml
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <docker.image.prefix>your-dockerhub-username</docker.image.prefix>
    </properties>
```

`docker.image.prefix` helps define image repository names consistently.

---

# Add JUnit Dependency

```xml
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

JUnit is used only for testing.

It is not needed inside the final Docker image.

---

# Add Surefire Plugin

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.2.5</version>
</plugin>
```

This plugin runs unit tests during:

```bash
mvn test
mvn package
```

---

# Add JAR Plugin

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.company.app.App</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

This makes the JAR executable.

---

# Step 6: Add Dockerfile

Create file:

```text
Dockerfile
```

Add:

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY target/maven-docker-app-1.0.0.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

# Explanation of Dockerfile

```dockerfile
FROM eclipse-temurin:17-jre
```

Uses Java 17 runtime image.

```dockerfile
WORKDIR /app
```

Sets working directory inside container.

```dockerfile
COPY target/... app.jar
```

Copies Maven-generated JAR into image.

```dockerfile
ENTRYPOINT
```

Runs the app when container starts.

---

# Step 7: Build JAR with Maven

Run:

```bash
mvn clean package
```

This command:

- Cleans previous builds
- Compiles code
- Runs tests
- Creates JAR file

Expected artifact:

```text
target/maven-docker-app-1.0.0.jar
```

---

# Step 8: Build Docker Image Manually

Run:

```bash
docker build -t your-dockerhub-username/maven-docker-app:1.0.0 .
```

Meaning:

- `docker build`: build image
- `-t`: tag image
- `username/app:version`: image name and version
- `.`: use current folder as build context

---

# Step 9: Run Docker Container

Run:

```bash
docker run --rm your-dockerhub-username/maven-docker-app:1.0.0
```

Expected output:

```text
Maven Docker App is running inside container.
```

This confirms the image works.

---

# Step 10: Push Image to Docker Hub

Login first:

```bash
docker login
```

Then push:

```bash
docker push your-dockerhub-username/maven-docker-app:1.0.0
```

Now the image is available in Docker Hub.

---

# Step 11: Pull and Verify Image

From another machine:

```bash
docker pull your-dockerhub-username/maven-docker-app:1.0.0
docker run --rm your-dockerhub-username/maven-docker-app:1.0.0
```

This verifies registry-based distribution.

---

# Maven Docker Plugin Concept

Instead of manually running Docker commands, Maven can trigger Docker build and push.

One commonly used plugin is:

```text
dockerfile-maven-plugin
```

Purpose:

- Build Docker image from Maven
- Tag image using Maven version
- Push image to registry
- Integrate containerization into Maven lifecycle

---

# dockerfile-maven-plugin Note

The `dockerfile-maven-plugin` is widely seen in older Maven-Docker examples.

In modern projects, teams may also use:

- Jib Maven Plugin
- Spotify dockerfile-maven-plugin
- Fabric8 Docker Maven Plugin
- Docker Buildx in CI/CD

The concept remains the same:

```text
Maven build → Docker image build → Registry push
```

---

# Step 12: Add dockerfile-maven-plugin

Inside `<plugins>`:

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.13</version>
    <configuration>
        <repository>${docker.image.prefix}/${project.artifactId}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

---

# Dockerfile with Build Argument

Update Dockerfile:

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

ARG JAR_FILE

COPY ${JAR_FILE} app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

This allows Maven plugin to pass JAR path dynamically.

---

# Step 13: Build Image Using Maven

Run:

```bash
mvn clean package dockerfile:build
```

This performs:

```text
Maven clean
Maven package
Docker image build
```

Expected image:

```text
your-dockerhub-username/maven-docker-app:1.0.0
```

---

# Step 14: Push Image Using Maven

Run:

```bash
mvn dockerfile:push
```

Before this, ensure:

```bash
docker login
```

has been completed.

The image will be pushed to Docker Hub.

---

# Step 15: Build and Push Together

A common command:

```bash
mvn clean package dockerfile:build dockerfile:push
```

This creates:

```text
JAR artifact
Docker image
Registry image
```

in one automated workflow.

---

# Complete pom.xml Structure

The POM contains:

```text
properties
dependencies
build
  plugins
    maven-surefire-plugin
    maven-jar-plugin
    dockerfile-maven-plugin
```

This integrates testing, packaging, and Docker image creation.

---

# Complete Build Flow

```text
mvn clean package dockerfile:build dockerfile:push
```

Steps executed:

1. Clean old target files
2. Compile Java code
3. Run unit tests
4. Package JAR
5. Build Docker image
6. Push image to registry

---

# Tagging Strategy

Good image tags include:

```text
1.0.0
1.0.1
latest
commit-sha
build-number
```

Example:

```bash
docker tag app:1.0.0 app:latest
```

Avoid using only `latest` in production because it is not precise.

---

# Docker Hub Image Format

Docker Hub format:

```text
username/image-name:tag
```

Example:

```text
alokmisra/maven-docker-app:1.0.0
```

---

# GitHub Container Registry Format

GHCR format:

```text
ghcr.io/username/image-name:tag
```

Example:

```text
ghcr.io/alokmisra/maven-docker-app:1.0.0
```

Login command:

```bash
docker login ghcr.io
```

---

# CI/CD Pipeline Example

```yaml
name: Maven Docker Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
```

---

# CI/CD Pipeline Continued

```yaml
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
```

---

# CI/CD Pipeline Continued

```yaml
      - name: Build Maven Package
        run: mvn clean package

      - name: Build Docker Image
        run: docker build -t your-dockerhub-username/maven-docker-app:1.0.0 .

      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
```

---

# CI/CD Pipeline Continued

```yaml
      - name: Push Docker Image
        run: docker push your-dockerhub-username/maven-docker-app:1.0.0
```

This automates build and registry publishing.

---

# Why Use Secrets in CI/CD?

Registry passwords should never be written directly in workflow files.

Use CI/CD secrets:

```text
DOCKER_USERNAME
DOCKER_PASSWORD
```

This protects credentials.

---

# Verification Commands

| Command | Purpose |
|---|---|
| `mvn clean package` | Build Java JAR |
| `docker build` | Build Docker image |
| `docker images` | List local images |
| `docker run` | Test container |
| `docker push` | Push to registry |
| `docker pull` | Verify registry image |

---

# Common Errors and Fixes

| Error | Cause | Solution |
|---|---|---|
| `COPY failed` | JAR not created | Run `mvn package` first |
| `no main manifest attribute` | Main class missing | Configure JAR plugin |
| Docker login failed | Wrong credentials | Use correct token/password |
| Image push denied | Repository permission issue | Check username/repository |
| Port not accessible | Port not exposed/mapped | Use `-p host:container` |

---

# Important Teaching Point

Maven creates the artifact.

Docker packages the artifact with runtime.

Registry distributes the packaged image.

Together, they enable:

```text
Build once, run anywhere
```

---

# Maven-Docker Integration Benefits

| Benefit | Explanation |
|---|---|
| Automation | Reduces manual steps |
| Consistency | Same JAR goes into image |
| Traceability | Image tag can match Maven version |
| CI/CD friendly | Easy pipeline integration |
| Deployment ready | Image can run anywhere Docker is available |

---

# When to Build Docker Image?

Best practice:

```text
Build Docker image only after Maven tests pass.
```

Recommended order:

```text
mvn clean test
mvn package
docker build
docker push
```

This prevents broken applications from becoming deployable images.

---

# Practical Verification Checklist

Students should verify:

- Maven project compiles successfully.
- Unit tests pass.
- JAR file is generated in `target`.
- Dockerfile correctly copies the JAR.
- Docker image builds successfully.
- Container runs and prints expected output.
- Image is tagged properly.
- Image can be pushed and pulled.

---

# Final Answer Summary

Maven and Docker integration automates the path from source code to deployable container image.

Maven compiles, tests, and packages the application into a JAR.

Docker packages the JAR with a Java runtime into an image.

Using `dockerfile-maven-plugin`, Maven can build and push Docker images as part of the build process.

---

# Short Exam-Style Answer

Maven and Docker integration is useful because Maven creates the Java artifact while Docker packages it into a portable runtime image. The process begins with `mvn clean package`, which compiles the code, runs tests, and creates a JAR. A Dockerfile then copies the JAR into a Java runtime image. Tools such as `dockerfile-maven-plugin` can automate Docker image creation and pushing from Maven using commands like `mvn clean package dockerfile:build dockerfile:push`. This ensures consistent, test-verified, and registry-ready deployments.

---

# Student Activity

Perform the following:

1. Create a Maven Java project.
2. Add `App.java`.
3. Add `AppTest.java`.
4. Configure `pom.xml`.
5. Run `mvn clean package`.
6. Write Dockerfile.
7. Build Docker image manually.
8. Run Docker container.
9. Configure dockerfile-maven-plugin.
10. Build and push image using Maven.

---

# End of Scenario 10

## Topic Covered

**Maven and Docker integration with registry push workflow**

---
