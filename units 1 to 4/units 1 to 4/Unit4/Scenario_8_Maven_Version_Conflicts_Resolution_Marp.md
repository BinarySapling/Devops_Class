---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 8: Maven Version Conflicts and Resolution"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven version conflicts and dependency resolution"
---

# Scenario 8  
## Maven Version Conflicts and Resolution

**Syllabus Coverage:**  
Version conflicts, nearest definition rule, dependency mediation, dependencyManagement, exclusions, dependency tree analysis.

---

# Question 8

Two frameworks in the same project require different versions of the same logging library, causing runtime conflicts.

As a DevOps engineer:

1. Explain how Maven handles version conflicts conceptually.  
2. Describe how you would identify the conflicting dependency.  
3. Show how to force the correct version using `dependencyManagement` in `pom.xml`.  
4. Explain additional strategies such as exclusions.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain Maven version conflict resolution.
- Understand the nearest-wins rule.
- Use `mvn dependency:tree`.
- Detect duplicate versions.
- Use `dependencyManagement` to enforce versions.
- Apply exclusions when needed.
- Build stable enterprise projects.

---

# Problem Understanding

The project uses:

```text
Framework A
Framework B
```

Both require:

```text
logging-library
```

But versions are different:

```text
Framework A → logging-library 1.2
Framework B → logging-library 2.0
```

This may cause runtime conflicts.

---

# Why Runtime Conflict Happens

If classes differ between versions:

- Methods may be missing
- APIs may have changed
- Unexpected behavior occurs
- Application may crash

Typical errors:

```text
NoSuchMethodError
ClassNotFoundException
LinkageError
```

---

# Dependency Conflict Graph

```text
My Project
 ├── Framework A
 │     └── logging-library:1.2
 └── Framework B
       └── logging-library:2.0
```

Maven must choose one version.

---

# How Maven Resolves Conflicts

Maven uses **dependency mediation**.

Main rule:

```text
Nearest definition wins
```

Meaning:

The version closest to your project in the dependency tree is selected.

---

# Nearest-Wins Example

```text
Project
 ├── A → log:1.2
 └── B → X → log:2.0
```

Selected version:

```text
1.2
```

Why?

Because `log:1.2` is closer to the project root.

---

# Same Depth Conflict

If two versions are at same depth:

```text
Project
 ├── A → log:1.2
 └── B → log:2.0
```

Then Maven often uses the dependency declared first.

This is why dependency order can matter.

---

# Why This Is Dangerous

Automatic selection may not be the version you want.

Possible issues:

- Old vulnerable library selected
- New framework incompatible with old version
- Hidden production failures
- Hard debugging

Therefore explicit control is preferred.

---

# Implementation Goal

We will:

1. Create a Maven project.
2. Simulate two frameworks with conflicting versions.
3. Inspect tree.
4. Force approved version.
5. Rebuild and verify.

---

# Step 1: Create Project Folder

```bash
mkdir conflict-demo
cd conflict-demo
```

---

# Step 2: Example pom.xml with Conflicting Dependencies

We simulate with common libraries.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

<modelVersion>4.0.0</modelVersion>

<groupId>com.demo</groupId>
<artifactId>conflict-demo</artifactId>
<version>1.0.0</version>
```

---

# Add Dependencies

```xml
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>3.2.5</version>
    </dependency>

    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.20.0</version>
    </dependency>

</dependencies>
</project>
```

This can create multiple logging paths.

---

# Step 3: Inspect Dependency Tree

Run:

```bash
mvn dependency:tree
```

Purpose:

- Shows selected versions
- Shows omitted versions
- Helps detect duplicates

---

# Example Output

```text
[INFO] +- spring-boot-starter-web:3.2.5
[INFO] |  \- spring-boot-starter-logging
[INFO] |     \- logback-classic:1.4.x
[INFO] \- log4j-core:2.20.0
```

Now two logging systems may coexist.

---

# Search Specific Dependency

```bash
mvn dependency:tree -Dincludes=org.apache.logging.log4j
```

or

```bash
mvn dependency:tree -Dincludes=ch.qos.logback
```

This narrows investigation.

---

# Common Real Conflict Types

| Conflict | Example |
|---|---|
| Different versions | log4j 2.17 vs 2.20 |
| Multiple frameworks | logback + log4j |
| API mismatch | old commons-lang |
| Security patch lag | vulnerable old version |

---

# Step 4: Use dependencyManagement

To force one version centrally:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.23.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

This sets the approved version.

---

# Meaning of dependencyManagement

This tells Maven:

> Whenever `log4j-core` is used, prefer version `2.23.1`.

It centralizes version governance.

Useful in enterprise multi-module builds.

---

# Updated pom.xml Structure

```xml
<project ...>

<dependencyManagement>
   ...
</dependencyManagement>

<dependencies>
   ...
</dependencies>

</project>
```

---

# Step 5: Rebuild and Verify

Run:

```bash
mvn clean package
```

Then:

```bash
mvn dependency:tree -Dincludes=org.apache.logging.log4j
```

Expected:

```text
log4j-core:2.23.1
```

---

# Step 6: Use Exclusions

If Framework A brings unwanted version:

```xml
<dependency>
    <groupId>com.vendor</groupId>
    <artifactId>framework-a</artifactId>
    <version>1.0</version>

    <exclusions>
        <exclusion>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

This blocks inherited version.

---

# Why Exclusions Help

Without exclusion:

```text
framework-a → old log4j
```

With exclusion:

```text
framework-a (without old log4j)
```

Then project can add approved version directly.

---

# Best Practice Strategy

Use both:

1. Exclude unsafe/old version.
2. Add approved version directly.
3. Control centrally via dependencyManagement.

This gives predictable builds.

---

# Example Final pom.xml Concept

```xml
<dependencyManagement>
   approved versions
</dependencyManagement>

<dependencies>
   framework-a with exclusions
   framework-b
   approved logging library
</dependencies>
```

---

# Step 7: Effective POM Check

Run:

```bash
mvn help:effective-pom
```

Purpose:

- Shows final merged configuration
- Confirms managed versions
- Useful in parent-child projects

---

# Step 8: Analyze Runtime Errors

Common signs of version conflict:

```text
NoSuchMethodError
NoClassDefFoundError
ClassCastException
SLF4J multiple bindings warning
```

These often indicate dependency mismatch.

---

# Logging Conflict Example

```text
SLF4J: Class path contains multiple SLF4J bindings
```

Meaning:

More than one logging backend exists.

Need to keep only one intended implementation.

---

# Commands Summary

| Command | Purpose |
|---|---|
| `mvn dependency:tree` | Full dependency graph |
| `mvn dependency:tree -Dincludes=...` | Filter dependency |
| `mvn clean package` | Rebuild project |
| `mvn help:effective-pom` | Final config |
| `mvn dependency:analyze` | Usage analysis |

---

# CI/CD Pipeline Example

```yaml
name: Conflict Check

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

---

# Enterprise Best Practices

| Practice | Benefit |
|---|---|
| Parent POM | Central governance |
| dependencyManagement | Fixed versions |
| Exclusions | Remove bad transitive libs |
| Security scans | Detect CVEs |
| Regular upgrades | Lower risk |

---

# Common Mistakes

| Mistake | Impact | Fix |
|---|---|---|
| Ignore warnings | Runtime failures | Inspect logs |
| Multiple logging frameworks | Duplicate bindings | Keep one |
| Force old version | Security risk | Use supported version |
| No dependency tree check | Hidden conflicts | Run analysis |
| Manual jar copying | Chaos | Use Maven only |

---

# Practical Verification Checklist

Students should verify:

- Dependency tree shows actual selected version.
- Conflicting libraries identified.
- dependencyManagement added.
- Exclusions applied if needed.
- Clean build succeeds.
- Runtime warnings disappear.

---

# Final Answer Summary

Maven handles version conflicts using dependency mediation, usually the nearest definition rule.

To identify conflicts, run:

```bash
mvn dependency:tree
```

To enforce the correct version, use:

```xml
<dependencyManagement>
```

For unwanted inherited versions, use:

```xml
<exclusions>
```

This creates stable and predictable enterprise builds.

---

# Short Exam-Style Answer

When multiple dependencies request different versions of the same library, Maven resolves the conflict using dependency mediation. Usually the version nearest to the project is selected. To inspect conflicts, use `mvn dependency:tree`. To force an approved version, define it inside `<dependencyManagement>` in `pom.xml`. If an older version is coming through another framework, it can be blocked using `<exclusions>`. These practices help avoid runtime errors and maintain consistent builds.

---

# Student Activity

Perform the following:

1. Create a Maven project.
2. Add two libraries with overlapping transitive dependencies.
3. Run `mvn dependency:tree`.
4. Identify conflicting versions.
5. Add `dependencyManagement`.
6. Force one approved version.
7. Add exclusion if needed.
8. Rebuild project.
9. Compare dependency tree before and after.
10. Explain nearest-wins rule.

---

# End of Scenario 8

## Topic Covered

**Maven version conflicts and dependency resolution**

---
