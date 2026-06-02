---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 4: Maven Compile vs Test Phase and Surefire Plugin"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven compile phase, test phase, and Surefire plugin"
---

# Scenario 4  
## Maven Compile vs Test Phase and Surefire Plugin

**Syllabus Coverage:**  
Maven lifecycle phases, `compile`, `test`, unit testing, Surefire plugin, automated build failure, and CI/CD testing.

---

# Question 4

A project successfully compiles but fails during the `test` phase because unit tests are broken.

As a DevOps engineer:

1. Explain the conceptual difference between the `compile` and `test` phases in Maven.  
2. Explain why successful compilation does not guarantee a successful build.  
3. Show how the Maven Surefire Plugin is used to execute automated tests during the build.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain the role of the `compile` phase.
- Explain the role of the `test` phase.
- Understand why unit tests are important.
- Configure JUnit in a Maven project.
- Configure Maven Surefire Plugin.
- Execute and debug unit test failures.
- Understand test failure impact in CI/CD pipelines.

---

# Problem Understanding

The project compiles successfully.

This means:

```text
Java syntax is correct.
Required classes are available.
Source code can be converted into bytecode.
```

However, the project fails during testing.

This means:

```text
The program may not behave as expected.
One or more unit tests are failing.
Business logic may be incorrect.
```

---

# Maven Lifecycle Recap

Common Maven lifecycle phases are:

```text
validate → compile → test → package → verify → install → deploy
```

When we run:

```bash
mvn test
```

Maven automatically runs:

```text
validate → compile → test
```

So testing happens only after successful compilation.

---

# What Happens in Compile Phase?

The `compile` phase converts Java source code into bytecode.

Source location:

```text
src/main/java
```

Output location:

```text
target/classes
```

Example command:

```bash
mvn compile
```

If the source code has syntax errors, this phase fails.

---

# Example Compile Phase

Input file:

```text
src/main/java/com/example/Calculator.java
```

Output file:

```text
target/classes/com/example/Calculator.class
```

Maven checks whether Java code is syntactically correct.

It does not fully check whether the logic is correct.

---

# What Happens in Test Phase?

The `test` phase runs unit tests.

Test source location:

```text
src/test/java
```

Test output location:

```text
target/test-classes
```

Test reports:

```text
target/surefire-reports
```

Example command:

```bash
mvn test
```

---

# Key Difference Between Compile and Test

| Phase | Main Purpose | Checks |
|---|---|---|
| `compile` | Converts source code to `.class` files | Syntax and type correctness |
| `test` | Executes unit tests | Functional correctness |
| `package` | Creates JAR/WAR | Final artifact creation |

A project can compile successfully but still fail tests.

---

# Why Compilation Success Is Not Enough

Compilation only confirms:

- No syntax errors
- Required classes exist
- Method calls are valid
- Java compiler can generate bytecode

Compilation does not confirm:

- Correct output
- Correct business logic
- Correct calculations
- Correct edge case handling
- Correct application behavior

---

# Simple Example

A calculator method may compile correctly:

```java
public int add(int a, int b) {
    return a - b;
}
```

This code compiles because syntax is valid.

But logically, it is wrong because addition should use:

```java
return a + b;
```

Unit testing catches such logical mistakes.

---

# Step 1: Create Maven Project Folder

Create a project folder:

```bash
mkdir maven-test-demo
cd maven-test-demo
```

This folder will contain the Maven project.

---

# Step 2: Create Maven Directory Structure

For Linux, macOS, or Git Bash:

```bash
mkdir -p src/main/java/com/example
mkdir -p src/test/java/com/example
```

For Windows Command Prompt:

```cmd
mkdir src\main\java\com\example
mkdir src\test\java\com\example
```

---

# Step 3: Create Calculator Class

Create file:

```text
src/main/java/com/example/Calculator.java
```

Add the following code:

```java
package com.example;

public class Calculator {
    public int add(int a, int b) {
        return a - b;
    }
}
```

This code has no syntax error but has logical error.

---

# Explanation of Calculator Code

```java
public int add(int a, int b)
```

This method is expected to add two numbers.

But the implementation is:

```java
return a - b;
```

This is logically incorrect.

Therefore, compilation will pass but unit testing should fail.

---

# Step 4: Create Unit Test Class

Create file:

```text
src/test/java/com/example/CalculatorTest.java
```

Add this code:

```java
package com.example;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class CalculatorTest {
    @Test
    public void testAdd() {
        Calculator calculator = new Calculator();
        assertEquals(5, calculator.add(2, 3));
    }
}
```

---

# Explanation of Unit Test

```java
@Test
```

This marks the method as a test case.

```java
assertEquals(5, calculator.add(2, 3));
```

This expects the output to be `5`.

But the method returns:

```text
2 - 3 = -1
```

So the test will fail.

---

# Step 5: Create pom.xml

Create file:

```text
pom.xml
```

This file controls Maven build behavior.

It contains:

- Project coordinates
- Java version
- Test dependency
- Surefire plugin

---

# Basic POM Configuration

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>maven-test-demo</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
```

---

# Add Java Version

```xml
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
```

Purpose:

- Defines Java version
- Ensures consistent compilation
- Avoids local machine mismatch

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

Purpose:

- Adds JUnit 5 testing framework
- Available only during test phase
- Not included in production artifact

---

# Add Surefire Plugin

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

Purpose:

- Detects test classes
- Executes unit tests
- Generates test reports
- Fails the build if tests fail

---

# Complete pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>maven-test-demo</artifactId>
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

# Step 6: Run Compile Phase

Run:

```bash
mvn compile
```

Expected result:

```text
BUILD SUCCESS
```

Why?

Because the code is syntactically correct.

Even though the logic is wrong, the compiler cannot detect this type of business logic error.

---

# Step 7: Run Test Phase

Run:

```bash
mvn test
```

Expected result:

```text
BUILD FAILURE
```

Possible output:

```text
expected: <5> but was: <-1>
Tests run: 1, Failures: 1
```

This confirms that testing catches logic errors.

---

# Step 8: Locate Test Reports

After running tests, Maven creates reports in:

```text
target/surefire-reports
```

Important files may include:

```text
com.example.CalculatorTest.txt
TEST-com.example.CalculatorTest.xml
```

These reports are useful for debugging and CI/CD logs.

---

# Step 9: Fix the Logical Error

Open:

```text
src/main/java/com/example/Calculator.java
```

Replace:

```java
return a - b;
```

with:

```java
return a + b;
```

Corrected class:

```java
package com.example;

public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}
```

---

# Step 10: Run Tests Again

Run:

```bash
mvn test
```

Expected output:

```text
Tests run: 1, Failures: 0, Errors: 0
BUILD SUCCESS
```

Now the project passes both compilation and testing.

---

# Step 11: Package After Tests Pass

Run:

```bash
mvn package
```

Maven executes:

```text
validate → compile → test → package
```

Expected artifact:

```text
target/maven-test-demo-1.0.0.jar
```

The JAR is created only after tests pass.

---

# Role of Surefire Plugin

The Surefire Plugin runs tests during the Maven `test` phase.

It automatically detects test classes with names such as:

```text
*Test.java
Test*.java
*Tests.java
*TestCase.java
```

Example:

```text
CalculatorTest.java
```

---

# Surefire Plugin in Build Automation

During CI/CD:

```bash
mvn clean test
```

or

```bash
mvn clean verify
```

causes Surefire to execute unit tests.

If any test fails:

```text
Build fails
Artifact is not released
Defective code is stopped early
```

---

# Maven Commands Used in This Scenario

| Command | Purpose |
|---|---|
| `mvn validate` | Validate project configuration |
| `mvn compile` | Compile main source code |
| `mvn test` | Run unit tests |
| `mvn package` | Create JAR after tests pass |
| `mvn clean test` | Clean old output and run tests |

---

# CI/CD Pipeline Example

```yaml
name: Maven Test Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  test:
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

      - name: Run Unit Tests
        run: mvn clean test
```

This ensures that broken logic is detected before packaging or deployment.

---

# Why This Matters in Real Projects

Without unit tests:

- Wrong code may be packaged
- Defects may reach production
- Customers may face errors
- Debugging becomes expensive
- CI/CD pipeline becomes unreliable

With Surefire and JUnit:

- Logic errors are caught early
- Build quality improves
- Developers get quick feedback
- Release confidence increases

---

# Compile Success vs Test Success

| Situation | Meaning |
|---|---|
| Compile success | Code syntax is correct |
| Test success | Code behavior is correct for tested cases |
| Compile failure | Code cannot be converted to bytecode |
| Test failure | Code compiles but gives wrong result |

Both compile and test success are needed for a reliable build.

---

# Common Test Failure Causes

| Cause | Example | Fix |
|---|---|---|
| Wrong logic | Addition method subtracts | Correct method |
| Wrong expected value | Expected 6 instead of 5 | Fix test expectation |
| Missing dependency | JUnit not added | Add JUnit dependency |
| Wrong test name | Test not detected | Use `*Test.java` naming |
| Plugin issue | Old Surefire version | Configure updated plugin |

---

# Important Teaching Point

A compiler checks language rules.

A unit test checks expected behavior.

Therefore:

```text
Compilation answers: Can the code run?
Testing answers: Does the code work correctly?
```

This is the central lesson of this scenario.

---

# Practical Verification Checklist

Students should verify:

- Project has correct Maven structure.
- `pom.xml` includes JUnit dependency.
- Surefire plugin is configured.
- `mvn compile` succeeds.
- `mvn test` fails initially.
- Test report is generated.
- Logic error is fixed.
- `mvn test` succeeds after correction.
- `mvn package` creates the JAR.

---

# Final Answer Summary

The `compile` phase checks whether source code can be converted into bytecode, while the `test` phase checks whether the code behaves correctly through unit tests.

A project may compile successfully but still fail during testing because logical errors are not detected by the compiler.

The Maven Surefire Plugin executes unit tests during the `test` phase and fails the build if any test fails.

---

# Short Exam-Style Answer

The `compile` phase in Maven checks the syntax and compiles source files from `src/main/java` into `.class` files. The `test` phase executes unit tests from `src/test/java` using tools such as JUnit and the Maven Surefire Plugin. Successful compilation does not guarantee correct logic; it only means the code is syntactically valid. If a method produces wrong output, unit tests can fail even after successful compilation. The Surefire Plugin runs these tests automatically and prevents faulty builds from moving forward.

---

# Student Activity

Perform the following:

1. Create a Maven project.
2. Write a calculator class with wrong logic.
3. Add JUnit dependency.
4. Configure Surefire Plugin.
5. Run `mvn compile`.
6. Observe build success.
7. Run `mvn test`.
8. Observe test failure.
9. Fix the logic.
10. Run `mvn test` and `mvn package` again.

---

# End of Scenario 4

## Topic Covered

**Maven compile phase, test phase, and Surefire Plugin**

---
