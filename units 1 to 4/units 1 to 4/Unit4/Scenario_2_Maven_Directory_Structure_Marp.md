---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 2: Maven Standard Directory Structure"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven directory structure"
---

# Scenario 2  
## Maven Standard Directory Structure

**Syllabus Coverage:**  
Maven directory structure, POM, source folders, test folders, resources, build automation, and CI compatibility.

---

# Question 2

A new intern creates a Java project but stores source files, test files, and resources in random folders. The CI server cannot detect the project properly.

As a DevOps engineer:

1. Explain the importance of Maven’s standard directory structure.  
2. Describe how you would reorganize the project into the correct Maven layout.  
3. Explain how you would verify that the project builds successfully.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain why Maven follows a standard project layout.
- Identify the correct location of source code, test code, and resources.
- Reorganize an incorrectly structured Java project.
- Create a valid `pom.xml`.
- Execute Maven commands to verify the build.
- Understand why CI servers prefer standard build structures.

---

# Problem Understanding

The intern has created a Java project without following Maven conventions.

Possible issues:

- Java files are placed in random folders.
- Test files are not inside the test directory.
- Resource files are mixed with source files.
- CI server cannot identify the project properly.
- Maven lifecycle commands may fail.

This problem is related to poor project structure.

---

# Why Directory Structure Matters

Maven follows the principle of:

> Convention over configuration

This means Maven expects files to be placed in predefined folders.

If the project follows Maven conventions, Maven automatically knows:

- Where source code is located
- Where test code is located
- Where resources are stored
- Where compiled files should be generated
- Where packaged artifacts should be placed

---

# Correct Maven Directory Structure

A standard Maven Java project should look like this:

```text
student-app/
│
├── pom.xml
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/example/App.java
    │   └── resources/
    │       └── application.properties
    │
    └── test/
        ├── java/
        │   └── com/example/AppTest.java
        └── resources/
            └── test.properties
```

---

# Meaning of Each Folder

| Folder | Purpose |
|---|---|
| `src/main/java` | Main application source code |
| `src/main/resources` | Application configuration files |
| `src/test/java` | Unit test source code |
| `src/test/resources` | Test-specific configuration files |
| `target` | Generated build output |
| `pom.xml` | Maven project configuration |

---

# Incorrect Project Structure Example

The intern may have created something like this:

```text
student-app/
│
├── App.java
├── AppTest.java
├── config.properties
├── libraries/
│   └── junit.jar
└── build-output/
```

Problems:

- Source code is not inside `src/main/java`.
- Test code is not inside `src/test/java`.
- Dependencies are copied manually.
- Maven cannot automatically detect files.
- CI server may fail during build execution.

---

# Corrected Project Structure

The project should be reorganized as:

```text
student-app/
│
├── pom.xml
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/example/App.java
    │   └── resources/
    │       └── config.properties
    └── test/
        └── java/
            └── com/example/AppTest.java
```

This structure allows Maven and CI tools to work properly.

---

# Step 1: Create Project Root Folder

Create a project folder:

```bash
mkdir student-app
cd student-app
```

This folder will contain:

- `pom.xml`
- `src` folder
- Maven build output after compilation

---

# Step 2: Create Maven Source Folders

For Linux, macOS, or Git Bash:

```bash
mkdir -p src/main/java/com/example
mkdir -p src/main/resources
mkdir -p src/test/java/com/example
mkdir -p src/test/resources
```

For Windows Command Prompt:

```cmd
mkdir src\main\java\com\example
mkdir src\main\resources
mkdir src\test\java\com\example
mkdir src\test\resources
```

---

# Step 3: Move Main Java File

Move the main application file:

```text
App.java
```

to:

```text
src/main/java/com/example/App.java
```

This is the correct location for production source code.

---

# Step 4: Add Package Declaration

Inside `App.java`, add the correct package declaration:

```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Maven directory structure is correct.");
    }

    public static String greet(String name) {
        return "Hello, " + name;
    }
}
```

---

# Explanation of App.java

```java
package com.example;
```

This package statement must match the folder path:

```text
com/example
```

```java
main()
```

This method starts the Java program.

```java
greet()
```

This method is added so that it can be tested through a unit test.

---

# Step 5: Move Test File

Move the test file:

```text
AppTest.java
```

to:

```text
src/test/java/com/example/AppTest.java
```

This is the correct Maven location for unit tests.

---

# Step 6: Write Unit Test Code

Add this code in `AppTest.java`:

```java
package com.example;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class AppTest {
    @Test
    public void testGreet() {
        assertEquals("Hello, Alok", App.greet("Alok"));
    }
}
```

---

# Explanation of Test Code

```java
@Test
```

Marks the method as a test case.

```java
assertEquals(expected, actual)
```

Compares expected output with actual output.

If both values match, the test passes.

If values do not match, Maven fails the build.

---

# Step 7: Move Resource Files

Move application configuration files to:

```text
src/main/resources
```

Example:

```text
src/main/resources/application.properties
```

Sample file:

```properties
app.name=Student Maven App
app.version=1.0.0
```

---

# Why Resources Are Kept Separately

Resources may include:

- `.properties` files
- `.xml` configuration files
- `.json` files
- static files
- logging configuration

Maven automatically copies files from:

```text
src/main/resources
```

to:

```text
target/classes
```

during the build.

---

# Step 8: Create pom.xml

Create a file named:

```text
pom.xml
```

at the root of the project.

The POM tells Maven:

- What the project is
- Which Java version to use
- Which dependencies are required
- Which plugins should run

---

# Basic pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>student-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

</project>
```

---

# Add Java Version

Add this section inside `pom.xml`:

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

Purpose:

- Ensures Java 17 is used
- Avoids compiler mismatch
- Maintains consistent encoding

---

# Add JUnit Dependency

Add this inside the POM:

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

This allows Maven to download JUnit automatically.

No manual copying of JAR files is required.

---

# Add Surefire Plugin

Add this inside the `<build>` section:

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

This plugin runs unit tests during Maven build execution.

---

# Complete pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>student-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
```

---

# Complete pom.xml Continued

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

# Complete pom.xml Continued

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
</project>
```

---

# Step 9: Validate Project Structure

Run:

```bash
mvn validate
```

Purpose:

- Checks whether the project is correctly configured
- Validates the POM file
- Ensures Maven can understand the project

If successful, Maven shows:

```text
BUILD SUCCESS
```

---

# Step 10: Compile the Project

Run:

```bash
mvn compile
```

Purpose:

- Compiles files from `src/main/java`
- Places compiled `.class` files in `target/classes`

Expected output:

```text
target/classes/com/example/App.class
```

---

# Step 11: Run Tests

Run:

```bash
mvn test
```

Purpose:

- Compiles test files from `src/test/java`
- Runs JUnit tests
- Generates test reports

Expected output:

```text
Tests run: 1, Failures: 0, Errors: 0
BUILD SUCCESS
```

---

# Step 12: Package the Project

Run:

```bash
mvn package
```

Purpose:

- Executes validate, compile, test, and package phases
- Creates final JAR file

Expected artifact:

```text
target/student-app-1.0.0.jar
```

---

# Step 13: Run the Application

Run:

```bash
java -cp target/student-app-1.0.0.jar com.example.App
```

Expected output:

```text
Maven directory structure is correct.
```

This confirms that the Maven project was reorganized and built successfully.

---

# Step 14: Check Generated Target Folder

After successful build, Maven creates:

```text
target/
├── classes/
├── generated-sources/
├── generated-test-sources/
├── maven-archiver/
├── surefire-reports/
├── test-classes/
└── student-app-1.0.0.jar
```

This folder contains compiled files, test reports, and packaged artifacts.

---

# Why CI Server Failed Earlier

The CI server failed because:

- It expected a Maven-standard project.
- Source files were not in expected locations.
- Test files were not in expected locations.
- Dependencies were manually handled.
- `pom.xml` was missing or incomplete.

CI tools rely on predictable structures for automation.

---

# CI-Compatible Build Command

A CI server can now execute:

```bash
mvn clean verify
```

This command:

- Removes old build files
- Validates project configuration
- Compiles source code
- Runs unit tests
- Packages and verifies the build

---

# Example CI Pipeline Step

```yaml
name: Maven Directory Structure Check

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

# Example CI Pipeline Continued

```yaml
      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build Project
        run: mvn clean verify
```

This CI pipeline works because the project now follows Maven conventions.

---

# Manual Structure vs Maven Structure

| Aspect | Random Structure | Maven Structure |
|---|---|---|
| Source location | Unclear | `src/main/java` |
| Test location | Unclear | `src/test/java` |
| Resources | Mixed with code | `src/main/resources` |
| Build output | Manual | `target` |
| CI support | Poor | Strong |
| Maintainability | Low | High |

---

# Common Mistakes

| Mistake | Impact | Correction |
|---|---|---|
| Keeping Java files in root folder | Maven cannot compile automatically | Move to `src/main/java` |
| Missing package declaration | Classpath issues | Match package with folder path |
| Tests outside test folder | Tests not executed | Move to `src/test/java` |
| Manual JAR libraries | Version mismatch | Use Maven dependencies |
| Missing POM | Maven cannot build | Create valid `pom.xml` |

---

# Teaching Explanation

The main idea is simple:

Maven does not search randomly for files.

It expects a fixed structure.

When the structure is correct, Maven can automatically:

- Compile source code
- Copy resources
- Run test cases
- Package the application
- Generate build reports
- Support CI/CD pipelines

---

# Practical Verification Checklist

Students should check:

- `pom.xml` is in the root folder.
- Main code is in `src/main/java`.
- Test code is in `src/test/java`.
- Resources are in `src/main/resources`.
- Package declarations match folder structure.
- `mvn validate` succeeds.
- `mvn compile` succeeds.
- `mvn test` succeeds.
- `mvn package` creates a JAR.

---

# Final Answer Summary

Maven’s standard directory structure is important because it allows Maven and CI tools to automatically identify source code, test code, resources, dependencies, and build outputs.

To solve the problem, the project should be reorganized into:

```text
src/main/java
src/main/resources
src/test/java
src/test/resources
```

A valid `pom.xml` should be created, and the project should be verified using:

```bash
mvn clean verify
```

---

# Short Exam-Style Answer

Maven follows a standard directory structure so that build activities can be automated without custom configuration. Main source code should be placed in `src/main/java`, test code in `src/test/java`, resources in `src/main/resources`, and project configuration in `pom.xml`. After reorganizing the project, the build can be verified using Maven commands such as `mvn validate`, `mvn compile`, `mvn test`, and `mvn package`. This enables consistent local and CI builds.

---

# Student Activity

Perform the following:

1. Create an incorrectly structured Java project.
2. Move files into Maven-standard folders.
3. Add correct package declarations.
4. Create a valid `pom.xml`.
5. Add JUnit dependency.
6. Run `mvn validate`.
7. Run `mvn compile`.
8. Run `mvn test`.
9. Run `mvn package`.
10. Submit screenshot of `BUILD SUCCESS`.

---

# End of Scenario 2

## Topic Covered

**Maven standard directory structure and build verification**

---
