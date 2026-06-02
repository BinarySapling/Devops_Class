---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 7: Transitive Dependencies and Excluding Unsafe Libraries"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven transitive dependencies and exclusion"
---

# Scenario 7  
## Transitive Dependencies and Excluding Unsafe Libraries

**Syllabus Coverage:**  
Transitive dependencies, dependency tree, exclusions, security remediation, Maven dependency management, enterprise risk control.

---

# Question 7

A project directly depends on **Library A**, but Library A automatically pulls **Library B** and **Library C**.

Later, a security issue is found in Library C.

As a DevOps engineer:

1. Explain the concept of **transitive dependencies**.  
2. Explain one benefit and one risk of transitive dependencies.  
3. Describe how you would inspect the dependency tree.  
4. Show how to exclude the unsafe dependency in Maven.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain direct vs transitive dependencies.
- Understand how Maven resolves dependency graphs.
- Use `mvn dependency:tree`.
- Identify risky inherited libraries.
- Exclude unsafe transitive dependencies.
- Replace excluded dependencies with safer versions.
- Understand software supply chain security basics.

---

# Problem Understanding

The project directly adds:

```text
Library A
```

But Maven automatically downloads:

```text
Library B
Library C
```

because they are required by Library A.

Now Library C has a security vulnerability.

Even though developers never added Library C directly, it is still part of the project.

---

# What Is a Direct Dependency?

A dependency explicitly written in `pom.xml`.

Example:

```xml
<dependency>
    <groupId>com.vendor</groupId>
    <artifactId>library-a</artifactId>
    <version>1.0.0</version>
</dependency>
```

This means the project intentionally uses Library A.

---

# What Is a Transitive Dependency?

A dependency required by another dependency.

If:

```text
Project → Library A
Library A → Library B
Library A → Library C
```

Then:

- Library A = direct dependency
- Library B = transitive dependency
- Library C = transitive dependency

Maven downloads them automatically.

---

# Dependency Graph Visualization

```text
My Project
   |
   └── Library A
         ├── Library B
         └── Library C
```

This dependency graph is built automatically by Maven.

---

# Why Maven Uses Transitive Dependencies

Many libraries need other libraries to function.

Example:

- Web framework may need logging library.
- Database library may need connection pool library.
- JSON library may need annotations library.

Instead of manually adding everything, Maven resolves them automatically.

---

# Benefit of Transitive Dependencies

## Benefit: Faster Development

Developers add only the main dependency.

Maven automatically downloads supporting libraries.

This saves time and reduces manual work.

Example:

```text
Add Spring Boot starter → many supporting libraries arrive automatically
```

---

# Risk of Transitive Dependencies

## Risk: Hidden Security Vulnerabilities

A vulnerable library may enter the project indirectly.

Developers may not notice it because it was never explicitly declared.

This creates:

- Security exposure
- Compliance risk
- Unexpected runtime issues
- Version conflicts

---

# Real-World Example

A project uses:

```text
library-a:1.0
```

which pulls:

```text
commons-io:2.4
```

Later, a known vulnerability is discovered in:

```text
commons-io:2.4
```

Even though the team never added `commons-io` directly, the project is affected.

---

# Implementation Goal

We will create a Maven project that depends on:

```text
Library A
```

and inspect its dependency tree.

Then we will:

1. Detect unsafe Library C.
2. Exclude Library C.
3. Add a safer version manually.

---

# Step 1: Create Project Folder

```bash
mkdir transitive-demo
cd transitive-demo
```

This is the Maven project root.

---

# Step 2: Create pom.xml with Example Dependency

We use a realistic example:

```text
Spring Boot starter web
```

because it pulls many transitive dependencies.

Create:

```text
pom.xml
```

---

# pom.xml Basic Structure

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.demo</groupId>
    <artifactId>transitive-demo</artifactId>
    <version>1.0.0</version>
```

---

# Add Dependency

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.5</version>
        </dependency>
    </dependencies>

</project>
```

This single dependency brings many transitive libraries.

---

# Step 3: Inspect Dependency Tree

Run:

```bash
mvn dependency:tree
```

Purpose:

- Shows all direct and transitive dependencies.
- Displays dependency hierarchy.
- Helps identify hidden libraries.

---

# Example Output

```text
com.demo:transitive-demo:jar:1.0.0
└── org.springframework.boot:spring-boot-starter-web:3.2.5
    ├── spring-boot-starter:3.2.5
    ├── spring-web:6.x
    ├── spring-webmvc:6.x
    └── jackson-databind:2.x
```

This confirms transitive dependency loading.

---

# Filter Specific Dependency

To search for a library:

```bash
mvn dependency:tree -Dincludes=com.fasterxml.jackson.core
```

This helps locate where a dependency came from.

Useful in large enterprise projects.

---

# Step 4: Simulate Unsafe Library C

Assume:

```text
jackson-databind:2.x
```

is identified as vulnerable.

It was not directly added.

It came through:

```text
spring-boot-starter-web
```

We must remove or replace it.

---

# How Exclusion Works

Maven allows excluding transitive dependencies.

We exclude the risky library from the parent dependency.

Syntax:

```xml
<exclusions>
    <exclusion>
        ...
    </exclusion>
</exclusions>
```

---

# Step 5: Exclude Unsafe Dependency

Update dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.2.5</version>

    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

This removes the unsafe transitive dependency.

---

# Meaning of Exclusion

This tells Maven:

> Use Spring Boot starter web, but do not bring `jackson-databind`.

So dependency graph changes from:

```text
A → B + C
```

to:

```text
A → B
```

---

# Step 6: Add Safe Replacement Version

After exclusion, manually add a safer version.

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.1</version>
</dependency>
```

Now the project uses an approved secure version.

---

# Updated pom.xml Concept

```xml
<dependencies>

    <!-- Parent library -->
    <dependency>
        ...
        <exclusions>
            ...
        </exclusions>
    </dependency>

    <!-- Safe replacement -->
    <dependency>
        ...
    </dependency>

</dependencies>
```

This is a common enterprise remediation pattern.

---

# Step 7: Recheck Dependency Tree

Run:

```bash
mvn dependency:tree
```

Expected result:

Unsafe version disappears.

Safe version appears.

This confirms remediation success.

---

# Step 8: Use Dependency Analysis

Run:

```bash
mvn dependency:analyze
```

Purpose:

- Finds unused dependencies
- Finds undeclared but used dependencies
- Helps cleanup projects

Useful after exclusions.

---

# Step 9: Build the Project

Run:

```bash
mvn clean package
```

Purpose:

- Validates POM
- Resolves updated dependencies
- Compiles code
- Packages application

Expected:

```text
BUILD SUCCESS
```

---

# Why This Matters in Security

Modern attacks often target software supply chains.

Risky libraries may enter through:

- Direct dependencies
- Transitive dependencies
- Old plugin versions
- Unmaintained packages

Therefore, dependency visibility is critical.

---

# Recommended Security Practices

| Practice | Benefit |
|---|---|
| Run `dependency:tree` | See hidden libraries |
| Use approved versions | Reduce vulnerabilities |
| Exclude unsafe transitive libs | Remove risky code |
| Use Parent POM | Centralized control |
| Run CI scans | Continuous security checks |

---

# Direct vs Transitive Dependency

| Type | Example |
|---|---|
| Direct | Written in your POM |
| Transitive | Pulled automatically |
| Optional | May not always be included |
| Excluded | Intentionally blocked |

---

# Example Graph After Fix

```text
My Project
   |
   └── Library A
         ├── Library B
         └── Library C (safe version)
```

Unsafe version removed.

---

# Common Commands

| Command | Purpose |
|---|---|
| `mvn dependency:tree` | Show full dependency tree |
| `mvn dependency:analyze` | Analyze dependency usage |
| `mvn clean package` | Build with updated dependencies |
| `mvn help:effective-pom` | Show final configuration |

---

# CI/CD Pipeline Example

```yaml
name: Dependency Security Check

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
```

---

# CI/CD Continued

```yaml
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: mvn clean package

      - name: Dependency Tree
        run: mvn dependency:tree
```

This gives visibility in automated pipelines.

---

# Common Mistakes

| Mistake | Impact | Fix |
|---|---|---|
| Ignore transitive libs | Hidden risks remain | Inspect tree |
| Exclude wrong dependency | Build failure | Check artifact coordinates |
| Remove dependency without replacement | Runtime issues | Add safe version |
| Never upgrade | Security exposure | Patch regularly |
| Duplicate versions | Conflict risk | Use dependencyManagement |

---

# Practical Verification Checklist

Students should verify:

- Dependency tree is generated.
- Hidden libraries are visible.
- Unsafe library identified.
- Exclusion added correctly.
- Safe version manually added.
- Project builds successfully.
- Dependency tree confirms fix.

---

# Final Answer Summary

Transitive dependencies are libraries automatically downloaded because another dependency needs them.

Their main benefit is convenience and reduced manual setup.

Their main risk is hidden vulnerabilities or version conflicts.

A DevOps engineer should inspect dependencies using:

```bash
mvn dependency:tree
```

Then exclude the unsafe library using `<exclusions>` and add a safe replacement version.

---

# Short Exam-Style Answer

A transitive dependency is a library indirectly added through another dependency. For example, if a project depends on Library A and Library A depends on Library C, then Library C becomes a transitive dependency. This saves time because developers do not need to manually add all supporting libraries. However, it also introduces risks such as hidden vulnerabilities. To inspect such dependencies, use `mvn dependency:tree`. Unsafe libraries can be removed using the `<exclusions>` tag in `pom.xml`, and a secure replacement version can be added explicitly.

---

# Student Activity

Perform the following:

1. Create a Maven project.
2. Add a starter dependency with many transitive libraries.
3. Run `mvn dependency:tree`.
4. Identify one transitive library.
5. Add exclusion for that library.
6. Add newer safe version manually.
7. Re-run `dependency:tree`.
8. Run `mvn clean package`.
9. Compare before and after trees.
10. Explain benefit and risk of transitive dependencies.

---

# End of Scenario 7

## Topic Covered

**Transitive dependencies and excluding unsafe libraries**

---
