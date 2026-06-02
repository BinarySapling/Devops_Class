---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 5: Maven Lifecycle for Compile, Test, and Package"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven package workflow"
---

# Scenario 5  
## One Command to Compile, Test, and Package

**Syllabus Coverage:**  
Maven lifecycle phases, `compile`, `test`, `package`, POM, JAR packaging, automated builds, and release preparation.

---

# Question 5

A startup wants one command that compiles code, runs tests, and packages a runnable JAR for deployment.

As a DevOps engineer:

1. Explain how Maven lifecycle phases `compile`, `test`, and `package` support this workflow.  
2. Describe how to implement this using the `mvn package` command.  
3. Explain how the generated JAR can be executed and verified.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain Maven lifecycle phase execution.
- Understand how `mvn package` works.
- Create a Maven-based Java application.
- Add unit testing support.
- Generate a JAR file.
- Run the generated JAR file.
- Understand why one standardized build command improves DevOps delivery.

---

# Problem Understanding

The startup wants a simple and repeatable build process.

Instead of running many manual commands, the team wants one command that can:

- Compile the source code
- Run unit tests
- Create a deployable JAR file

The Maven command for this requirement is:

```bash
mvn package
```

---

# Why One Build Command Is Important

A single standardized build command helps because:

- Every developer follows the same process.
- CI/CD pipelines become easier.
- Manual mistakes are reduced.
- Testing is not skipped.
- Packaging becomes consistent.
- Releases become more reliable.

This is a key principle in DevOps automation.

---

# Maven Lifecycle Concept

Maven lifecycle phases run in a sequence.

Important phases are:

```text
validate → compile → test → package → verify → install → deploy
```

If we run a later phase, Maven automatically runs all earlier required phases.

Therefore:

```bash
mvn package
```

automatically executes:

```text
validate → compile → test → package
```

---

# Meaning of Each Phase

| Phase | Purpose |
|---|---|
| `validate` | Checks project configuration |
| `compile` | Compiles main source code |
| `test` | Runs unit tests |
| `package` | Creates JAR/WAR file |
| `verify` | Performs additional quality checks |
| `install` | Installs artifact in local repository |
| `deploy` | Uploads artifact to remote repository |

---

# Focus of This Scenario

This scenario mainly focuses on:

```text
compile → test → package
```

The startup wants the application to be built, tested, and packaged using one command.

The command is:

```bash
mvn package
```

---

# Implementation Overview

We will implement:

1. Maven project folder
2. Standard directory structure
3. Java application class
4. Unit test class
5. `pom.xml`
6. JUnit dependency
7. Surefire plugin
8. Jar plugin for executable JAR
9. Build command
10. JAR execution command

---

# Step 1: Create Project Folder

Create the project folder:

```bash
mkdir startup-maven-app
cd startup-maven-app
```

This is the root directory of the Maven project.

It will contain:

```text
pom.xml
src/
target/
```

---

# Step 2: Create Maven Directory Structure

For Linux, macOS, or Git Bash:

```bash
mkdir -p src/main/java/com/startup
mkdir -p src/test/java/com/startup
```

For Windows Command Prompt:

```cmd
mkdir src\main\java\com\startup
mkdir src\test\java\com\startup
```

---

# Step 3: Create Application Class

Create file:

```text
src/main/java/com/startup/App.java
```

Add this code:

```java
package com.startup;

public class App {
    public static void main(String[] args) {
        System.out.println("Startup Maven App is running.");
    }

    public static String getStatus() {
        return "READY";
    }
}
```

---

# Explanation of App.java

```java
package com.startup;
```

This defines the package of the class.

```java
main()
```

This is the entry point of the application.

```java
getStatus()
```

This method is used for unit testing.

The method should return:

```text
READY
```

---

# Step 4: Create Unit Test Class

Create file:

```text
src/test/java/com/startup/AppTest.java
```

Add this code:

```java
package com.startup;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class AppTest {
    @Test
    public void testStatus() {
        assertEquals("READY", App.getStatus());
    }
}
```

---

# Explanation of Unit Test

```java
@Test
```

Marks the method as a test case.

```java
assertEquals("READY", App.getStatus());
```

Checks whether the actual output of `getStatus()` is equal to the expected value.

If the result is not `"READY"`, the test fails.

---

# Step 5: Create pom.xml

Create file:

```text
pom.xml
```

The POM file defines the project and controls the Maven build.

It includes:

- Project coordinates
- Java version
- Dependencies
- Plugins
- Packaging type

---

# Basic Project Coordinates

```xml
<groupId>com.startup</groupId>
<artifactId>startup-maven-app</artifactId>
<version>1.0.0</version>
<packaging>jar</packaging>
```

Explanation:

- `groupId`: organization or package identity
- `artifactId`: application name
- `version`: release version
- `packaging`: output will be a JAR file

---

# Complete pom.xml Part 1

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.startup</groupId>
    <artifactId>startup-maven-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
```

---

# Complete pom.xml Part 2

```xml
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
```

Purpose:

- Uses Java 17
- Keeps source and target Java version consistent
- Sets UTF-8 encoding

---

# Complete pom.xml Part 3

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

Purpose:

- Adds JUnit 5
- Enables unit testing
- Keeps testing library out of production classpath

---

# Complete pom.xml Part 4

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>
```

The Surefire Plugin runs unit tests during the `test` phase.

---

# Complete pom.xml Part 5

```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.startup.App</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

The JAR plugin adds the main class information.

---

# Why Main Class Is Needed

To run a JAR directly using:

```bash
java -jar target/startup-maven-app-1.0.0.jar
```

the JAR must know the main class.

This is configured using:

```xml
<mainClass>com.startup.App</mainClass>
```

Without this, Java may show:

```text
no main manifest attribute
```

---

# Complete pom.xml Full Version

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.startup</groupId>
    <artifactId>startup-maven-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
```

---

# Complete pom.xml Full Version Continued

```xml
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
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

# Complete pom.xml Full Version Continued

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
```

---

# Complete pom.xml Full Version Continued

```xml
                            <mainClass>com.startup.App</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

This POM is ready for compiling, testing, and packaging the application.

---

# Step 6: Run Maven Package Command

Run:

```bash
mvn package
```

This single command performs:

```text
validate → compile → test → package
```

The startup team does not need to manually run compile, test, and JAR creation separately.

---

# Step 7: What Happens Internally?

When `mvn package` runs:

1. Maven reads `pom.xml`.
2. Maven validates project configuration.
3. Maven downloads required dependencies.
4. Maven compiles source code.
5. Maven compiles test code.
6. Maven runs unit tests.
7. Maven creates the JAR file.

---

# Step 8: Expected Build Output

If everything is correct, Maven shows:

```text
BUILD SUCCESS
```

Expected generated artifact:

```text
target/startup-maven-app-1.0.0.jar
```

This JAR file is the deployable application artifact.

---

# Step 9: Run the JAR File

Run:

```bash
java -jar target/startup-maven-app-1.0.0.jar
```

Expected output:

```text
Startup Maven App is running.
```

This confirms the JAR was packaged correctly and is executable.

---

# Step 10: Verify Test Reports

Maven generates test reports in:

```text
target/surefire-reports
```

This folder contains:

```text
com.startup.AppTest.txt
TEST-com.startup.AppTest.xml
```

These files help developers and CI tools understand test results.

---

# Step 11: Clean and Package Again

Run:

```bash
mvn clean package
```

Difference:

- `clean` removes the old `target` directory.
- `package` creates a fresh build.

This is preferred for CI/CD because it avoids old build artifacts.

---

# Why `mvn clean package` Is Better for CI

In CI/CD, clean builds are preferred.

Reason:

- Avoids stale files
- Ensures fresh compilation
- Gives reliable results
- Prevents hidden dependency on old outputs

Recommended CI command:

```bash
mvn clean package
```

---

# If Tests Fail During Package

If the unit test fails, then:

```bash
mvn package
```

will stop at the `test` phase.

The JAR will not be created.

This ensures broken code is not packaged for deployment.

---

# Example of Test Failure

If `App.getStatus()` returns:

```java
return "FAILED";
```

but test expects:

```java
assertEquals("READY", App.getStatus());
```

Then Maven output will show:

```text
Tests run: 1, Failures: 1
BUILD FAILURE
```

---

# How to Skip Tests Temporarily

Sometimes developers may package without running tests:

```bash
mvn package -DskipTests
```

However, this should be avoided in production pipelines.

Skipping tests may allow defective code to reach deployment.

Use only for emergency or local experimentation.

---

# Recommended Commands

| Command | Use |
|---|---|
| `mvn compile` | Only compile code |
| `mvn test` | Compile and run tests |
| `mvn package` | Compile, test, and package |
| `mvn clean package` | Clean old files and package fresh |
| `mvn clean verify` | Stronger CI validation |

---

# CI/CD Pipeline Example

```yaml
name: Maven Package Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  package:
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

      - name: Build and Package
        run: mvn clean package
```

This automatically compiles, tests, and packages the application after each push.

---

# Manual Process vs Maven Package

| Activity | Manual Method | Maven Method |
|---|---|---|
| Compile | `javac` manually | `mvn compile` |
| Run tests | Manual test execution | `mvn test` |
| Create JAR | `jar` command manually | `mvn package` |
| Dependency handling | Manual copying | Automatic |
| Repeatability | Low | High |

---

# Real-World DevOps Benefit

Using `mvn clean package` gives the startup:

- Fast feedback
- Consistent builds
- Automated testing
- Deployable artifacts
- Easier CI/CD integration
- Reduced human error
- Better release confidence

This is why Maven is widely used in Java DevOps workflows.

---

# Practical Verification Checklist

Students should verify:

- Project follows Maven directory structure.
- `pom.xml` contains correct metadata.
- JUnit dependency is added.
- Surefire plugin is configured.
- JAR plugin contains main class.
- `mvn package` gives `BUILD SUCCESS`.
- JAR file exists in `target`.
- `java -jar` runs the application.

---

# Final Answer Summary

Maven lifecycle phases support the startup’s requirement by executing build steps in order. The `compile` phase compiles source code, the `test` phase runs unit tests, and the `package` phase creates the final JAR file.

The command:

```bash
mvn package
```

automatically runs all required earlier phases and creates a deployable artifact.

For a clean and reliable build, the recommended command is:

```bash
mvn clean package
```

---

# Short Exam-Style Answer

Maven supports one-command build automation through its lifecycle. When `mvn package` is executed, Maven first validates the project, compiles the source code, runs unit tests, and then packages the application into a JAR file. This avoids manual compilation, testing, and JAR creation. To implement this, the project should follow Maven’s standard directory structure, include a valid `pom.xml`, add required dependencies such as JUnit, configure plugins like Surefire and Jar Plugin, and run `mvn clean package` to generate the deployable JAR.

---

# Student Activity

Perform the following:

1. Create a Maven project.
2. Add `App.java`.
3. Add `AppTest.java`.
4. Create `pom.xml`.
5. Add JUnit dependency.
6. Configure Surefire Plugin.
7. Configure Jar Plugin with main class.
8. Run `mvn package`.
9. Run generated JAR using `java -jar`.
10. Submit screenshot of successful build and application output.

---

# End of Scenario 5

## Topic Covered

**Maven lifecycle: compile, test, and package using one command**

---
