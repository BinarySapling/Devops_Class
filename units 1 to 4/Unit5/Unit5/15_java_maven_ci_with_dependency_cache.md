# Scenario 15: Java Maven CI with Dependency Cache

## 1. Scenario

You are working on a project named `java-maven-ci-demo`. The task is to implement GitHub Actions for **Java Maven build**. This scenario focuses on **JDK 17 and Maven cache** and helps students understand a real CI/CD use case through actual YAML implementation.

---

## 2. Learning Objectives

After completing this scenario, students will be able to:

- Create a valid workflow under `.github/workflows/`.
- Identify the correct trigger for the given automation need.
- Configure jobs, runners, and steps.
- Use actions and shell commands correctly.
- Debug common workflow errors.

---

## 3. Required Tools

- GitHub account
- Git installed locally
- Code editor such as VS Code
- Basic command-line knowledge

---

## 4. Project Structure

```text
java-maven-ci-demo/
└── .github/
    └── workflows/
        └── maven-ci.yml
```

---


## Project Structure

```text
java-maven-ci-demo/
├── pom.xml
└── src/
    ├── main/
    │   └── java/
    │       └── App.java
    └── test/
        └── java/
            └── AppTest.java
```

## Workflow File

```yaml
name: Java Maven CI

on:
  push:
  pull_request:

jobs:
  maven-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Setup Java JDK
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17
          cache: maven

      - name: Build and test with Maven
        run: mvn clean test
```

## Sample `pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>java-maven-ci-demo</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

## Step-by-Step Implementation

1. Create Maven project structure.
2. Add `pom.xml`.
3. Add Java source and test files.
4. Add workflow file.
5. Push to GitHub.
6. Check Actions logs.

## Expected Output

```text
BUILD SUCCESS
```

## Explanation

`actions/setup-java@v4` installs the required JDK. `cache: maven` caches Maven dependencies for faster future builds.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Wrong Maven structure | Tests not found | Follow `src/main/java` and `src/test/java` |
| Missing `pom.xml` | Maven build fails | Add valid POM |
| Java version mismatch | Compilation fails | Match JDK and compiler version |


---

## Student Submission Checklist

Students should submit:

1. GitHub repository link.
2. Screenshot of workflow YAML file.
3. Screenshot of successful workflow run.
4. Short explanation of trigger, job, runner, and steps.
5. Error encountered and how it was fixed.

---

## Viva Questions

1. What is the purpose of this workflow?
2. Which trigger is used and why?
3. Which runner is used in this scenario?
4. What are the key steps in the workflow?
5. What common error can occur in this scenario?
