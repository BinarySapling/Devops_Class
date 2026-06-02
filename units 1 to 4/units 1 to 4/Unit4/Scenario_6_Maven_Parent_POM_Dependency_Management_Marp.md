---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 6: Parent POM and Dependency Management in Maven"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven Parent POM and dependency management"
---

# Scenario 6  
## Parent POM and Dependency Management

**Syllabus Coverage:**  
Parent POM, dependency management, multi-service projects, dependency version standardization, plugin management, and enterprise build consistency.

---

# Question 6

A large enterprise has 20 microservices, each with separate POM files and duplicate dependency versions.

This causes maintenance problems whenever a library upgrade is needed.

As a DevOps engineer:

1. Explain the role of a **Parent POM**.  
2. Explain how **dependency management** helps standardize versions.  
3. Describe how you would implement a centralized Parent POM for all services.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain why Parent POM is useful in enterprise projects.
- Understand dependency duplication problems.
- Use `<dependencyManagement>` to centralize versions.
- Understand the difference between `<dependencies>` and `<dependencyManagement>`.
- Create a parent Maven project.
- Connect child microservices to the Parent POM.
- Standardize Java version, dependency versions, and plugin versions.

---

# Problem Understanding

The enterprise has 20 microservices.

Each service has its own `pom.xml`.

Common problems are:

- Same dependency repeated in every service.
- Different versions of the same library.
- Difficult upgrades.
- Runtime version conflicts.
- Inconsistent Java versions.
- Inconsistent plugin versions.
- More maintenance effort.

---

# Example Problem

Service A uses:

```xml
<version>2.17.1</version>
```

Service B uses:

```xml
<version>2.20.0</version>
```

Service C uses:

```xml
<version>2.21.1</version>
```

for the same dependency.

This creates inconsistency across the enterprise.

---

# Why This Is a Serious Issue

Inconsistent dependency versions may cause:

- Runtime errors
- Security vulnerabilities
- Different behavior in different services
- Difficult debugging
- Delayed upgrades
- More testing effort
- CI/CD instability

A centralized version strategy is needed.

---

# What Is a Parent POM?

A **Parent POM** is a Maven POM file that contains common configuration shared by multiple projects.

It can define:

- Java version
- Encoding
- Common dependency versions
- Plugin versions
- Repository configuration
- Build settings
- Organization-wide standards

Child projects inherit these settings.

---

# Parent POM Analogy

Think of the Parent POM as a university academic regulation document.

Each department may run different courses, but all follow common rules such as:

- Attendance policy
- Examination policy
- Grading policy
- Academic calendar

Similarly, each microservice has its own code, but follows common Maven rules from the Parent POM.

---

# What Is Dependency Management?

`dependencyManagement` is used to define dependency versions centrally.

It does not automatically add the dependency to every child project.

It only says:

> If a child project uses this dependency, use this version.

This avoids repeating dependency versions in every child POM.

---

# Dependencies vs Dependency Management

| Section | Purpose |
|---|---|
| `<dependencies>` | Actually adds dependency to the project |
| `<dependencyManagement>` | Defines dependency version for child projects |
| `<pluginManagement>` | Defines plugin versions for child projects |

This distinction is very important.

---

# Example Without Dependency Management

Each microservice repeats the version:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.2.5</version>
</dependency>
```

Problem:

- Same version appears in many files.
- Upgrade requires editing 20 POM files.
- Mistakes are likely.

---

# Example With Dependency Management

Parent POM defines:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.5</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Child POM uses dependency without version.

---

# Child POM After Dependency Management

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Maven automatically uses the version from the Parent POM.

This reduces duplication and improves consistency.

---

# Implementation Goal

We will create:

```text
enterprise-parent/
│
├── pom.xml
├── user-service/
│   └── pom.xml
└── order-service/
    └── pom.xml
```

The Parent POM will define shared configuration.

Child services will inherit from it.

---

# Step 1: Create Parent Project Folder

Create the main enterprise folder:

```bash
mkdir enterprise-parent
cd enterprise-parent
```

This folder will contain the Parent POM and child services.

---

# Step 2: Create Child Service Folders

Create two sample microservices:

```bash
mkdir user-service
mkdir order-service
```

In a real enterprise, there may be 20 services.

Here, we use two services for demonstration.

---

# Step 3: Parent POM Packaging Type

The Parent POM should use:

```xml
<packaging>pom</packaging>
```

Why?

Because the parent itself does not produce a JAR.

It mainly manages common build configuration.

---

# Step 4: Create Parent pom.xml

Create file:

```text
enterprise-parent/pom.xml
```

Add the following configuration:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.enterprise</groupId>
    <artifactId>enterprise-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
```

---

# Parent pom.xml Continued

```xml
    <modules>
        <module>user-service</module>
        <module>order-service</module>
    </modules>

    <properties>
        <java.version>17</java.version>
        <junit.version>5.10.2</junit.version>
        <slf4j.version>2.0.13</slf4j.version>
        <maven.compiler.plugin.version>3.11.0</maven.compiler.plugin.version>
        <maven.surefire.plugin.version>3.2.5</maven.surefire.plugin.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
```

---

# Explanation of Properties

| Property | Meaning |
|---|---|
| `java.version` | Common Java version |
| `junit.version` | Common JUnit version |
| `slf4j.version` | Common logging library version |
| `maven.compiler.plugin.version` | Compiler plugin version |
| `maven.surefire.plugin.version` | Test plugin version |

Instead of changing many files, we update one property.

---

# Parent dependencyManagement

```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>

            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>${slf4j.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

---

# Meaning of dependencyManagement

This section centralizes dependency versions.

It means:

- JUnit version is controlled by parent.
- SLF4J version is controlled by parent.
- Child projects do not need to repeat versions.
- All services use consistent versions.

Important: Dependencies are not automatically added unless child POM declares them.

---

# Parent pluginManagement

```xml
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>${maven.compiler.plugin.version}</version>
                    <configuration>
                        <source>${java.version}</source>
                        <target>${java.version}</target>
                    </configuration>
                </plugin>
```

---

# Parent pluginManagement Continued

```xml
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>${maven.surefire.plugin.version}</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

This centralizes plugin versions and configuration.

---

# Step 5: Create user-service pom.xml

Create file:

```text
user-service/pom.xml
```

Add parent reference:

```xml
<parent>
    <groupId>com.enterprise</groupId>
    <artifactId>enterprise-parent</artifactId>
    <version>1.0.0</version>
</parent>
```

This connects the child service to the Parent POM.

---

# user-service pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.enterprise</groupId>
        <artifactId>enterprise-parent</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>user-service</artifactId>
    <packaging>jar</packaging>
```

---

# user-service pom.xml Continued

```xml
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

Notice that dependency versions are not written here.

---

# Step 6: Create order-service pom.xml

Create file:

```text
order-service/pom.xml
```

Add similar configuration:

```xml
<parent>
    <groupId>com.enterprise</groupId>
    <artifactId>enterprise-parent</artifactId>
    <version>1.0.0</version>
</parent>
```

The service inherits dependency and plugin versions from the parent.

---

# order-service pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.enterprise</groupId>
        <artifactId>enterprise-parent</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>order-service</artifactId>
    <packaging>jar</packaging>
```

---

# order-service pom.xml Continued

```xml
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

Again, no repeated version numbers are required.

---

# Step 7: Create Source Code for user-service

Create folders:

```bash
mkdir -p user-service/src/main/java/com/enterprise/user
mkdir -p user-service/src/test/java/com/enterprise/user
```

For Windows Command Prompt:

```cmd
mkdir user-service\\src\\main\\java\\com\\enterprise\\user
mkdir user-service\\src\\test\\java\\com\\enterprise\\user
```

---

# UserService.java

Create file:

```text
user-service/src/main/java/com/enterprise/user/UserService.java
```

Code:

```java
package com.enterprise.user;

public class UserService {
    public String getServiceName() {
        return "User Service";
    }
}
```

---

# UserServiceTest.java

Create file:

```text
user-service/src/test/java/com/enterprise/user/UserServiceTest.java
```

Code:

```java
package com.enterprise.user;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class UserServiceTest {
    @Test
    public void testServiceName() {
        UserService service = new UserService();
        assertEquals("User Service", service.getServiceName());
    }
}
```

---

# Step 8: Create Source Code for order-service

Create folders:

```bash
mkdir -p order-service/src/main/java/com/enterprise/order
mkdir -p order-service/src/test/java/com/enterprise/order
```

For Windows Command Prompt:

```cmd
mkdir order-service\\src\\main\\java\\com\\enterprise\\order
mkdir order-service\\src\\test\\java\\com\\enterprise\\order
```

---

# OrderService.java

Create file:

```text
order-service/src/main/java/com/enterprise/order/OrderService.java
```

Code:

```java
package com.enterprise.order;

public class OrderService {
    public String getServiceName() {
        return "Order Service";
    }
}
```

---

# OrderServiceTest.java

Create file:

```text
order-service/src/test/java/com/enterprise/order/OrderServiceTest.java
```

Code:

```java
package com.enterprise.order;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class OrderServiceTest {
    @Test
    public void testServiceName() {
        OrderService service = new OrderService();
        assertEquals("Order Service", service.getServiceName());
    }
}
```

---

# Step 9: Build All Services from Parent

From the parent folder, run:

```bash
mvn clean package
```

Maven builds all modules listed in:

```xml
<modules>
    <module>user-service</module>
    <module>order-service</module>
</modules>
```

Expected result:

```text
BUILD SUCCESS
```

---

# Step 10: Build Only One Service

To build only user-service:

```bash
mvn -pl user-service clean package
```

To build order-service:

```bash
mvn -pl order-service clean package
```

This is useful in large projects where building all services takes more time.

---

# Step 11: Check Dependency Tree

Run:

```bash
mvn dependency:tree
```

For a specific service:

```bash
mvn -pl user-service dependency:tree
```

This shows which dependency versions are being used.

It helps verify that child services are using parent-managed versions.

---

# Step 12: Check Effective POM

Run:

```bash
mvn help:effective-pom
```

For a specific service:

```bash
mvn -pl user-service help:effective-pom
```

This shows the final POM after inheritance.

It helps students see what configuration the child inherited from the parent.

---

# Step 13: Updating a Dependency Version

Suppose the enterprise wants to update JUnit from:

```text
5.10.2
```

to:

```text
5.11.0
```

Only update this in the Parent POM:

```xml
<junit.version>5.11.0</junit.version>
```

All child services automatically use the updated version.

---

# Step 14: Why This Reduces Maintenance

Without Parent POM:

```text
Update 20 service POM files manually.
```

With Parent POM:

```text
Update only one parent POM property.
```

This reduces:

- Manual effort
- Version mismatch
- Upgrade mistakes
- Security patch delay
- Maintenance cost

---

# Parent POM Benefits

| Benefit | Explanation |
|---|---|
| Centralized version control | One place to manage versions |
| Consistency | All services use same standards |
| Reduced duplication | Child POMs become smaller |
| Easier upgrades | Change once, apply everywhere |
| Better governance | Enterprise build rules are enforced |
| CI/CD stability | Pipelines use predictable configuration |

---

# Parent POM vs Child POM

| Parent POM | Child POM |
|---|---|
| Defines shared rules | Defines service-specific needs |
| Uses `packaging=pom` | Usually uses `packaging=jar` |
| Contains dependency management | Declares actual dependencies |
| Contains plugin management | Uses plugins as required |
| Controls versions | Inherits versions |

---

# Dependency Management Important Point

This section:

```xml
<dependencyManagement>
```

does not automatically add dependencies.

Child projects must still declare:

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
</dependency>
```

The version comes from the parent.

---

# Plugin Management Important Point

This section:

```xml
<pluginManagement>
```

does not always automatically execute plugins.

It mainly controls plugin versions and default configuration.

Child modules can use the plugin, and Maven applies the parent-managed version.

---

# CI/CD Pipeline Example

```yaml
name: Enterprise Maven Build

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
```

---

# CI/CD Pipeline Continued

```yaml
      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build All Services
        run: mvn clean verify
```

This builds all child modules using the Parent POM.

---

# Common Mistakes

| Mistake | Impact | Solution |
|---|---|---|
| Parent packaging is `jar` | Parent build behaves incorrectly | Use `pom` packaging |
| Child does not declare parent | No inheritance | Add `<parent>` section |
| Versions repeated in child POM | Duplication remains | Move versions to parent |
| Dependency only in dependencyManagement | Dependency not added | Declare dependency in child |
| Module name mismatch | Build fails | Match folder name in `<module>` |

---

# Practical Verification Checklist

Students should verify:

- Parent POM uses `<packaging>pom</packaging>`.
- Child POMs include `<parent>` section.
- Versions are centralized in parent properties.
- Dependencies in child POM do not repeat versions.
- `mvn clean package` builds all modules.
- `mvn dependency:tree` shows correct versions.
- `mvn help:effective-pom` confirms inheritance.

---

# Final Answer Summary

A Parent POM is used to centralize common Maven configuration across multiple projects. It helps an enterprise standardize dependency versions, plugin versions, Java version, encoding, and build rules.

Dependency management allows the parent to define versions once, while child projects declare dependencies without repeating version numbers.

This makes upgrades easier, reduces conflicts, and improves consistency across all microservices.

---

# Short Exam-Style Answer

A Parent POM provides common configuration that can be inherited by multiple Maven projects. In an enterprise with many microservices, it prevents duplication by centralizing dependency versions, plugin versions, Java version, and build rules. The `<dependencyManagement>` section defines dependency versions in one place, while child projects declare only the dependencies they need. To implement this, create a parent project with `<packaging>pom</packaging>`, add child services under `<modules>`, define common versions in `<properties>`, and connect each child using the `<parent>` section.

---

# Student Activity

Perform the following:

1. Create an `enterprise-parent` folder.
2. Create `user-service` and `order-service` modules.
3. Write Parent POM with `packaging=pom`.
4. Add modules in Parent POM.
5. Add dependency management for JUnit and SLF4J.
6. Add plugin management for Compiler and Surefire plugins.
7. Connect child POMs to Parent POM.
8. Add source and test classes.
9. Run `mvn clean package`.
10. Run `mvn dependency:tree` and explain version inheritance.

---

# End of Scenario 6

## Topic Covered

**Parent POM and Dependency Management in Maven**

---
