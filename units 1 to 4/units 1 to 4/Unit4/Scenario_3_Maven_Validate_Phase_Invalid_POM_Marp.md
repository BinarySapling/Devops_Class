---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 3: Maven Validate Phase and Invalid POM Debugging"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven validate phase and POM correction"
---

# Scenario 3  
## Maven Validate Phase and Invalid POM Debugging

**Syllabus Coverage:**  
Maven lifecycle phases, `validate`, POM, XML correctness, project metadata, automated builds, and CI/CD failure analysis.

---

# Question 3

During CI/CD execution, the pipeline stops at the `validate` phase due to an invalid `pom.xml`.

Developers are confused because no code was compiled yet.

As a DevOps engineer:

1. Explain the purpose of the `validate` phase in Maven’s lifecycle.  
2. Explain why Maven stops before compilation.  
3. Describe how you would debug and fix the POM so the pipeline can continue.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain the purpose of Maven’s `validate` phase.
- Understand why validation happens before compilation.
- Identify common `pom.xml` errors.
- Debug Maven build failures.
- Correct a broken POM file.
- Verify the build using Maven lifecycle commands.

---

# Problem Understanding

The CI/CD pipeline failed at:

```bash
mvn validate
```

This means Maven did not even reach:

```text
compile
test
package
verify
install
deploy
```

The failure occurred because Maven could not properly read or validate the project configuration.

---

# What is the Maven Build Lifecycle?

A Maven lifecycle is a sequence of build phases.

Common phases are:

```text
validate → compile → test → package → verify → install → deploy
```

Each phase has a specific purpose.

When a later phase is executed, Maven automatically runs all previous phases.

---

# Purpose of the Validate Phase

The `validate` phase checks whether the project is correct and whether all necessary information is available.

It verifies:

- Whether `pom.xml` is present
- Whether the POM has valid XML syntax
- Whether required Maven coordinates are present
- Whether project configuration is readable
- Whether Maven can understand the project structure

---

# Why No Code Was Compiled

Code compilation happens during the `compile` phase.

However, Maven must first understand the project configuration.

If `pom.xml` is invalid, Maven does not know:

- Project name
- Project version
- Packaging type
- Dependencies
- Plugins
- Java version
- Build instructions

Therefore, Maven stops before compilation.

---

# Simple Analogy

Think of Maven like a teacher checking an exam answer sheet.

Before checking the answer:

- Student name must be present
- Roll number must be present
- Subject code must be correct
- Pages must be readable

Similarly, Maven checks the POM before compiling code.

If the POM is wrong, Maven cannot proceed.

---

# Example of Invalid pom.xml

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>invalid-app</artifactId>
    <version>1.0.0</version>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
        </dependency>
    </dependencies>
```

Problem:

```text
The <project> tag is not closed.
```

---

# Common POM Errors

| Error Type | Example |
|---|---|
| XML syntax error | Missing closing tag |
| Missing coordinates | Missing `groupId`, `artifactId`, or `version` |
| Wrong dependency format | Missing dependency version |
| Invalid plugin section | Plugin placed outside `<build>` |
| Wrong nesting | `<dependencies>` placed inside `<properties>` |
| Typing mistake | `<dependecies>` instead of `<dependencies>` |

---

# Common Maven Error Message

A common error may look like this:

```text
[FATAL] Non-readable POM
[FATAL] The project com.example:invalid-app:1.0.0 has 1 error
[FATAL] Non-parseable POM pom.xml
```

This means Maven cannot parse the `pom.xml` file.

---

# Step 1: Open the Project Root

First, check whether `pom.xml` exists in the project root.

Correct location:

```text
invalid-app/
│
├── pom.xml
└── src/
    └── main/
        └── java/
```

Wrong location:

```text
invalid-app/
└── src/
    └── pom.xml
```

Maven expects `pom.xml` in the root directory.

---

# Step 2: Run Maven Validate Locally

Run:

```bash
mvn validate
```

Purpose:

- Reproduce the same error locally
- Confirm whether the issue is with the project
- Avoid debugging only on CI server

If the issue appears locally, the problem is in the project configuration.

---

# Step 3: Read the Error Message Carefully

Example:

```text
Non-parseable POM
expected end tag </project>
```

This tells us:

- Maven cannot read the XML
- A closing tag is missing
- The issue is inside `pom.xml`

Do not jump directly to source code debugging.

---

# Step 4: Check XML Structure

Every opening tag must have a closing tag.

Example:

```xml
<dependencies>
    ...
</dependencies>
```

Correct XML rule:

```text
Opening tag + content + closing tag
```

Example:

```xml
<artifactId>demo-app</artifactId>
```

---

# Step 5: Correct Basic POM Structure

A valid Maven POM should begin like this:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

</project>
```

---

# Step 6: Add Required Maven Coordinates

Every Maven project should normally define:

```xml
<groupId>com.example</groupId>
<artifactId>validated-app</artifactId>
<version>1.0.0</version>
<packaging>jar</packaging>
```

Meaning:

- `groupId`: organization or package group
- `artifactId`: project name
- `version`: project release version
- `packaging`: output type

---

# Step 7: Create Correct pom.xml

Use this corrected POM:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>validated-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
```

---

# Correct pom.xml Continued

```xml
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

</project>
```

This POM is now readable and valid for a simple Java project.

---

# Step 8: Add Application Source Code

Create folder:

```bash
mkdir -p src/main/java/com/example
```

For Windows Command Prompt:

```cmd
mkdir src\main\java\com\example
```

Create file:

```text
src/main/java/com/example/App.java
```

---

# App.java

```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("POM validation completed successfully.");
    }
}
```

This simple application helps verify that Maven can proceed beyond the `validate` phase.

---

# Step 9: Run Validate Again

Run:

```bash
mvn validate
```

Expected output:

```text
BUILD SUCCESS
```

This confirms:

- POM is readable
- XML syntax is correct
- Maven can understand the project

---

# Step 10: Compile the Project

Run:

```bash
mvn compile
```

Purpose:

- Maven now proceeds after validation
- Java source code is compiled
- `.class` files are generated

Expected output:

```text
target/classes/com/example/App.class
```

---

# Step 11: Package the Project

Run:

```bash
mvn package
```

Purpose:

- Executes lifecycle phases up to package
- Creates a JAR file

Expected output:

```text
target/validated-app-1.0.0.jar
```

This proves the original validation problem is solved.

---

# Step 12: Use Effective POM for Debugging

Run:

```bash
mvn help:effective-pom
```

Purpose:

- Shows the final POM used by Maven
- Includes inherited configuration
- Helps identify parent POM and plugin settings
- Useful in large enterprise projects

This is helpful when the POM seems correct but Maven behaves differently.

---

# Step 13: Use Debug Mode

Run:

```bash
mvn -X validate
```

Purpose:

- Shows detailed debug output
- Helps trace configuration issues
- Useful for complex CI/CD failures

Use debug mode when normal error messages are not enough.

---

# Step 14: CI Pipeline Correction

Earlier CI command:

```bash
mvn validate
```

failed because of invalid POM.

After correction, the CI pipeline can use:

```bash
mvn clean verify
```

This command validates, compiles, tests, packages, and verifies the project.

---

# Example GitHub Actions Pipeline

```yaml
name: Validate Maven Project

on:
  push:
    branches: [ "main" ]

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
```

---

# GitHub Actions Pipeline Continued

```yaml
      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Validate Maven Project
        run: mvn validate

      - name: Build Project
        run: mvn clean verify
```

This ensures POM validation is checked before full build execution.

---

# Why Validate is Important in CI/CD

The `validate` phase prevents invalid projects from moving ahead.

It avoids wasting CI resources on:

- Broken project configuration
- Missing metadata
- Invalid XML
- Incorrect dependency declarations
- Improper build setup

It fails early and saves time.

---

# Difference Between Validate and Compile

| Phase | Purpose |
|---|---|
| `validate` | Checks project configuration |
| `compile` | Converts `.java` files into `.class` files |
| `test` | Runs unit tests |
| `package` | Creates JAR or WAR file |
| `verify` | Performs additional checks |

Validation is about configuration, while compilation is about source code.

---

# Practical Debugging Checklist

When `mvn validate` fails, check:

- Is `pom.xml` in the root directory?
- Is the XML syntax correct?
- Are all tags properly closed?
- Are `groupId`, `artifactId`, and `version` present?
- Are dependencies written correctly?
- Are plugins placed inside `<build><plugins>`?
- Does the project use the correct parent POM?
- Does Maven run locally?

---

# Common Errors and Solutions

| Error | Cause | Solution |
|---|---|---|
| Non-parseable POM | XML syntax error | Fix missing or wrong tag |
| Missing artifactId | Required metadata missing | Add artifactId |
| Dependency version missing | Version not defined | Add version or use dependencyManagement |
| Plugin not found | Wrong plugin coordinates | Correct groupId/artifactId/version |
| Parent POM missing | Parent not available | Fix parent version or repository |

---

# Implementation Summary

To fix the problem:

1. Locate `pom.xml`.
2. Run `mvn validate`.
3. Read the Maven error message.
4. Fix XML syntax.
5. Add required project coordinates.
6. Define Java version.
7. Run `mvn validate` again.
8. Run `mvn compile`.
9. Run `mvn package`.
10. Update CI pipeline command.

---

# Final Answer Summary

The `validate` phase checks whether the Maven project is correct before compilation begins.

Maven stops before compiling because an invalid `pom.xml` prevents it from understanding the project structure, dependencies, plugins, and packaging instructions.

The solution is to fix the POM syntax, add required Maven coordinates, validate the project locally, and then rerun the CI pipeline.

---

# Short Exam-Style Answer

The `validate` phase is the first important phase of the Maven build lifecycle. It checks whether the project is correctly configured and whether the `pom.xml` is valid. If the POM is invalid, Maven cannot identify project metadata, dependencies, plugins, or packaging type, so it stops before compilation. To solve this issue, the DevOps engineer should inspect the error message, correct the XML syntax and project coordinates in `pom.xml`, run `mvn validate`, and then execute `mvn clean verify` in the CI pipeline.

---

# Student Activity

Perform the following:

1. Create a Maven project.
2. Intentionally remove the closing `</project>` tag.
3. Run `mvn validate`.
4. Observe the error message.
5. Fix the POM.
6. Run `mvn validate` again.
7. Run `mvn compile`.
8. Run `mvn package`.
9. Capture screenshot of both failure and success.
10. Explain why validation happens before compilation.

---

# End of Scenario 3

## Topic Covered

**Maven validate phase and invalid POM debugging**

---
