---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 1: Why Build Tools Exist and Automated Maven Builds"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven build tools"
---

# Scenario 1  
## Why Build Tools Exist and Automated Maven Builds

**Syllabus Coverage:**  
Build tools, automated builds, Maven POM, directory structure, lifecycle phases, dependency handling, and standardized releases.

---

# Question 1

A software company notices that developers are manually compiling source code, copying dependencies, and packaging applications differently on each machine, causing inconsistent builds.

As a DevOps engineer:

1. Explain why build tools like **Apache Maven** exist.  
2. Describe how you would implement an automated Maven build process to standardize releases across all systems.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain the need for automated build tools.
- Identify problems caused by manual builds.
- Understand the role of Maven in Java project automation.
- Create a basic Maven project structure.
- Write a basic `pom.xml`.
- Execute Maven lifecycle commands.
- Generate a deployable JAR file consistently.

---

# Conceptual Background

A **build tool** automates the process of converting source code into a deployable software artifact.

In Java projects, this usually includes:

- Compiling `.java` files into `.class` files
- Downloading required dependencies
- Running unit tests
- Packaging the application into a `.jar` or `.war`
- Installing or deploying the artifact

---

# Why Build Tools Exist

Build tools exist because manual build processes are:

- Time-consuming
- Error-prone
- Difficult to repeat
- Different across developer machines
- Hard to integrate with CI/CD pipelines

Without a build tool, every developer may follow a different process, leading to inconsistent results.

---

# Problems in the Given Scenario

In the company:

- Developers compile code manually.
- Dependencies are copied manually.
- JAR files are created differently.
- Some builds work on one system but fail on another.
- There is no single standard build command.
- Release quality depends on individual developer practices.

This creates the problem of **“works on my machine”**.

---

# How Maven Solves the Problem

Apache Maven solves these issues by providing:

- A standard project structure
- A central configuration file called `pom.xml`
- Automatic dependency management
- Standard build lifecycle phases
- Plugin-based automation
- Repeatable build commands
- CI/CD compatibility

---

# Role of POM in Maven

The **Project Object Model (POM)** is the heart of a Maven project.

The file name is:

```xml
pom.xml
```

It defines:

- Project name
- Project version
- Packaging type
- Java version
- Dependencies
- Plugins
- Build configuration

---

# Standard Maven Directory Structure

A typical Maven project follows this structure:

```text
my-maven-app/
│
├── pom.xml
└── src/
    ├── main/
    │   └── java/
    │       └── com/example/App.java
    └── test/
        └── java/
            └── com/example/AppTest.java
```

This structure allows Maven to automatically identify source code and test code.

---

# Step 1: Create Project Folder

Create a new project folder:

```bash
mkdir my-maven-app
cd my-maven-app
```

This folder will contain the Maven project files.

---

# Step 2: Create Maven Directory Structure

Create source and test directories:

```bash
mkdir -p src/main/java/com/example
mkdir -p src/test/java/com/example
```

On Windows Command Prompt, use:

```cmd
mkdir src\main\java\com\example
mkdir src\test\java\com\example
```

---

# Step 3: Create Java Application File

Create the file:

```text
src/main/java/com/example/App.java
```

Add the following code:

```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Automated Maven Build Successful!");
    }

    public static int add(int a, int b) {
        return a + b;
    }
}
```

---

# Explanation of Java Code

```java
public static void main(String[] args)
```

This is the starting point of the Java application.

```java
System.out.println(...)
```

This prints output on the screen.

```java
public static int add(int a, int b)
```

This method is added so that we can test the application using unit testing later.

---

# Step 4: Create Unit Test File

Create the file:

```text
src/test/java/com/example/AppTest.java
```

Add the following code:

```java
package com.example;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class AppTest {
    @Test
    public void testAdd() {
        assertEquals(5, App.add(2, 3));
    }
}
```

---

# Explanation of Unit Test

```java
@Test
```

This tells Maven and JUnit that this method is a test case.

```java
assertEquals(5, App.add(2, 3));
```

This checks whether the actual output of `App.add(2, 3)` is equal to `5`.

If the output is correct, the test passes.

---

# Step 5: Create pom.xml

Create a file named:

```text
pom.xml
```

Add the following Maven configuration:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-maven-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

</project>
```

---

# Explanation of Basic POM

```xml
<groupId>com.example</groupId>
```

Defines the organization or package group.

```xml
<artifactId>my-maven-app</artifactId>
```

Defines the project or application name.

```xml
<version>1.0.0</version>
```

Defines the current version of the application.

```xml
<packaging>jar</packaging>
```

Tells Maven to create a JAR file.

---

# Step 6: Add Java Version and JUnit Dependency

Update `pom.xml`:

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>

<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

# Explanation of Dependency

```xml
<dependency>
```

This tells Maven to download an external library.

```xml
<scope>test</scope>
```

This means JUnit is required only for testing and should not be included in the final production artifact.

Maven downloads this dependency automatically from Maven Central.

---

# Step 7: Add Maven Surefire Plugin

Add this plugin inside the `<build>` section:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
```

---

# Purpose of Surefire Plugin

The Maven Surefire Plugin is used to run unit tests.

When we execute:

```bash
mvn test
```

Maven automatically runs all test cases from:

```text
src/test/java
```

If any test fails, the build fails.

---

# Complete pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-maven-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>
```

---

# Complete pom.xml Continued

```xml
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>
        </plugins>
    </build>
</project>
```

---

# Step 8: Validate the Project

Run:

```bash
mvn validate
```

Purpose:

- Checks whether the project is correct
- Validates `pom.xml`
- Ensures required configuration is available

If the POM has syntax errors, Maven stops here.

---

# Step 9: Compile the Project

Run:

```bash
mvn compile
```

Purpose:

- Compiles source code from `src/main/java`
- Creates `.class` files
- Stores compiled files in the `target/classes` folder

Expected folder:

```text
target/classes/com/example/App.class
```

---

# Step 10: Run Unit Tests

Run:

```bash
mvn test
```

Purpose:

- Compiles test files
- Executes unit tests
- Uses Maven Surefire Plugin

Expected result:

```text
Tests run: 1, Failures: 0, Errors: 0
```

---

# Step 11: Package the Application

Run:

```bash
mvn package
```

Purpose:

- Runs previous phases automatically
- Compiles code
- Runs tests
- Creates JAR file

Expected output:

```text
target/my-maven-app-1.0.0.jar
```

---

# Maven Lifecycle Flow

When we run:

```bash
mvn package
```

Maven internally executes:

```text
validate → compile → test → package
```

This means developers do not need to run each step manually.

Maven ensures the same sequence is followed every time.

---

# Step 12: Run the Generated JAR

Run:

```bash
java -cp target/my-maven-app-1.0.0.jar com.example.App
```

Expected output:

```text
Automated Maven Build Successful!
```

This confirms that the application was built successfully.

---

# Step 13: Clean and Rebuild

Run:

```bash
mvn clean package
```

Purpose:

- `clean` deletes the old `target` folder
- `package` rebuilds the project from scratch

This ensures that the build does not depend on old files.

---

# Why This Standardizes the Build

After Maven automation, every developer uses the same command:

```bash
mvn clean package
```

This ensures:

- Same lifecycle
- Same dependencies
- Same compiler version
- Same test execution process
- Same packaging process
- Same final artifact

---

# Manual Build vs Maven Build

| Aspect | Manual Build | Maven Build |
|---|---|---|
| Compilation | Manually done | Automated |
| Dependency download | Manual copying | Automatic |
| Testing | Often skipped | Automated |
| Packaging | Manual JAR creation | Automated |
| Repeatability | Low | High |
| CI/CD support | Difficult | Easy |

---

# Implementation in CI/CD Pipeline

A CI/CD pipeline can use this command:

```bash
mvn clean verify
```

Purpose:

- Cleans old builds
- Validates project
- Compiles code
- Runs tests
- Packages artifact
- Performs verification checks

This creates confidence before release.

---

# Example GitHub Actions Workflow

```yaml
name: Maven Build

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

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

# GitHub Actions Workflow Continued

```yaml
      - name: Build with Maven
        run: mvn clean package

      - name: Run Tests
        run: mvn test
```

This pipeline automatically builds and tests the project whenever code is pushed to the `main` branch.

---

# Practical Verification Checklist

Students should verify:

- `pom.xml` exists in the root folder.
- Source code is inside `src/main/java`.
- Test code is inside `src/test/java`.
- `mvn validate` runs successfully.
- `mvn compile` creates `.class` files.
- `mvn test` executes unit tests.
- `mvn package` creates a JAR file.
- JAR file runs successfully.

---

# Common Errors and Fixes

| Error | Possible Reason | Solution |
|---|---|---|
| `mvn not recognized` | Maven not installed or PATH missing | Install Maven or use Maven wrapper |
| Compilation error | Java syntax issue | Fix source code |
| Test failure | Wrong expected output | Correct test or code |
| Dependency not found | Wrong dependency coordinates | Check groupId, artifactId, version |
| Invalid POM | XML syntax error | Validate `pom.xml` |

---

# Final Answer Summary

Build tools like Maven exist to automate and standardize software builds.

In this scenario, Maven solves the problem by:

- Providing a standard directory structure
- Managing dependencies automatically
- Running tests consistently
- Packaging the application in the same way
- Supporting CI/CD automation

The final standardized command is:

```bash
mvn clean package
```

---

# Short Exam-Style Answer

Apache Maven should be introduced because it automates compilation, dependency management, testing, packaging, and release preparation. It removes manual errors and ensures that every developer and CI server follows the same build process. To implement it, the project should be arranged using Maven’s standard directory structure, a `pom.xml` should be created with project metadata, dependencies, Java version, and plugins, and the build should be executed using commands such as `mvn clean package` or `mvn clean verify`.

---

# Student Activity

Perform the following tasks:

1. Create the Maven project structure.
2. Add `App.java`.
3. Add `AppTest.java`.
4. Create `pom.xml`.
5. Run `mvn validate`.
6. Run `mvn compile`.
7. Run `mvn test`.
8. Run `mvn package`.
9. Execute the generated JAR.
10. Submit screenshot of successful build output.

---

# End of Scenario 1

## Topic Covered

**Why build tools exist and how Maven automates builds**

---
