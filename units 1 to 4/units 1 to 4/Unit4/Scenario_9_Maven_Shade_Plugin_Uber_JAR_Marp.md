---
marp: true
theme: default
paginate: true
size: 16:9
title: "Scenario 9: Maven Shade Plugin and Uber JAR Creation"
author: "Alok Misra"
description: "Detailed conceptual and implementation-based solution for Maven Shade Plugin and executable uber JAR"
---

# Scenario 9  
## Maven Shade Plugin and Uber JAR Creation

**Syllabus Coverage:**  
Maven Shade Plugin, uber JAR, executable JAR, dependency packaging, Maven lifecycle, `package` phase, deployment readiness.

---

# Question 9

A team wants to distribute a single executable JAR containing the application and all required libraries so users do not need separate dependencies.

As a DevOps engineer:

1. Explain the purpose of the **Maven Shade Plugin**.  
2. Explain what an **uber JAR** is.  
3. Describe how you would configure Maven Shade Plugin to generate an executable uber JAR.  
4. Show how to build and run the final JAR.

---

# Expected Learning Outcomes

After solving this scenario, students should be able to:

- Explain the need for an uber JAR.
- Understand the role of Maven Shade Plugin.
- Differentiate normal JAR and shaded JAR.
- Configure Shade Plugin in `pom.xml`.
- Define the application main class.
- Package dependencies inside one JAR.
- Execute the final JAR using `java -jar`.

---

# Problem Understanding

The team wants to distribute only one file:

```text
application.jar
```

The users should not need to separately download:

- Dependency JARs
- External libraries
- Classpath configuration
- Supporting runtime libraries

This requirement can be solved by creating an **uber JAR**.

---

# What Is a Normal JAR?

A normal JAR usually contains:

- Application `.class` files
- Resource files
- Manifest file

But it may not contain external dependencies.

Example:

```text
my-app.jar
```

This JAR may fail if dependencies are missing from the classpath.

---

# Problem with Normal JAR

If the application uses an external library such as Gson:

```java
import com.google.gson.Gson;
```

A normal JAR may not include Gson inside it.

Running the JAR may produce:

```text
ClassNotFoundException
NoClassDefFoundError
```

because dependency classes are missing.

---

# What Is an Uber JAR?

An **uber JAR** is a single JAR file that contains:

- Application classes
- Required dependency classes
- Resource files
- Manifest with main class

It is also called:

```text
fat JAR
shaded JAR
all-in-one JAR
```

---

# Uber JAR Analogy

A normal JAR is like a student carrying only the answer sheet.

An uber JAR is like carrying:

- Answer sheet
- Question paper
- Pen
- Calculator
- Required documents

Everything needed is available in one package.

---

# Purpose of Maven Shade Plugin

The Maven Shade Plugin is used to:

- Package application classes
- Include all required dependencies
- Create an executable JAR
- Relocate packages if needed
- Avoid dependency classpath problems

It runs during the Maven `package` phase.

---

# Normal JAR vs Uber JAR

| Feature | Normal JAR | Uber JAR |
|---|---|---|
| Contains application code | Yes | Yes |
| Contains dependencies | Usually no | Yes |
| Easy to run directly | Sometimes | Yes |
| Requires classpath setup | Often yes | No |
| Suitable for simple distribution | Limited | Strong |

---

# Implementation Goal

We will create a Maven project that:

1. Uses an external dependency.
2. Builds an executable uber JAR.
3. Runs using one command.
4. Does not require external classpath setup.

External dependency used:

```text
Gson
```

---

# Step 1: Create Project Folder

Create the project folder:

```bash
mkdir shade-demo
cd shade-demo
```

This folder will contain:

```text
pom.xml
src/
target/
```

---

# Step 2: Create Maven Directory Structure

For Linux, macOS, or Git Bash:

```bash
mkdir -p src/main/java/com/example
```

For Windows Command Prompt:

```cmd
mkdir src\main\java\com\example
```

---

# Step 3: Create Application Class

Create file:

```text
src/main/java/com/example/App.java
```

This file will use Gson dependency.

---

# App.java

```java
package com.example;

import com.google.gson.Gson;
import java.util.HashMap;
import java.util.Map;

public class App {
    public static void main(String[] args) {
        Map<String, String> data = new HashMap<>();
        data.put("app", "Shade Demo");
        data.put("status", "Uber JAR Created Successfully");

        Gson gson = new Gson();
        String json = gson.toJson(data);

        System.out.println(json);
    }
}
```

---

# Explanation of App.java

```java
import com.google.gson.Gson;
```

This imports an external library.

```java
Map<String, String> data = new HashMap<>();
```

This creates sample data.

```java
gson.toJson(data)
```

This converts Java object data into JSON format.

The application needs Gson at runtime.

---

# Step 4: Create pom.xml

Create file:

```text
pom.xml
```

The POM will include:

- Project coordinates
- Java version
- Gson dependency
- Maven Shade Plugin configuration

---

# Basic POM

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>shade-demo</artifactId>
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

- Ensures Java 17 compilation
- Keeps build consistent across machines
- Avoids compiler mismatch

---

# Add Gson Dependency

```xml
    <dependencies>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.10.1</version>
        </dependency>
    </dependencies>
```

This dependency is required by the application.

Without Shade Plugin, Gson may not be inside the final JAR.

---

# Add Maven Shade Plugin

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.5.1</version>
            </plugin>
        </plugins>
    </build>
</project>
```

This adds Shade Plugin to the build.

---

# Configure Shade Plugin Execution

The plugin must run during the `package` phase.

```xml
<executions>
    <execution>
        <phase>package</phase>
        <goals>
            <goal>shade</goal>
        </goals>
    </execution>
</executions>
```

Meaning:

When `mvn package` runs, the Shade Plugin creates the uber JAR.

---

# Add Main Class to Manifest

To run with:

```bash
java -jar file.jar
```

the JAR needs a main class.

Configure:

```xml
<transformers>
    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
        <mainClass>com.example.App</mainClass>
    </transformer>
</transformers>
```

---

# Complete Shade Plugin Configuration

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.5.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.example.App</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

---

# Complete pom.xml Part 1

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>shade-demo</artifactId>
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

    <dependencies>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.10.1</version>
        </dependency>
    </dependencies>
```

---

# Complete pom.xml Part 3

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.5.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
```

---

# Complete pom.xml Part 4

```xml
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.example.App</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

---

# Step 5: Build the Uber JAR

Run:

```bash
mvn clean package
```

Maven executes:

```text
validate → compile → test → package
```

During `package`, the Shade Plugin creates the uber JAR.

---

# Step 6: Check Target Folder

After build, check:

```text
target/
```

Expected files:

```text
shade-demo-1.0.0.jar
original-shade-demo-1.0.0.jar
```

Meaning:

- `shade-demo-1.0.0.jar` = shaded uber JAR
- `original-shade-demo-1.0.0.jar` = original normal JAR

---

# Step 7: Run the Uber JAR

Run:

```bash
java -jar target/shade-demo-1.0.0.jar
```

Expected output:

```json
{"app":"Shade Demo","status":"Uber JAR Created Successfully"}
```

This confirms the JAR includes Gson and runs independently.

---

# Step 8: Verify Contents of JAR

Run:

```bash
jar tf target/shade-demo-1.0.0.jar
```

This lists files inside the JAR.

You should see:

```text
com/example/App.class
com/google/gson/
META-INF/MANIFEST.MF
```

This confirms dependency classes are packaged inside.

---

# Step 9: Check Manifest File

Run:

```bash
jar xf target/shade-demo-1.0.0.jar META-INF/MANIFEST.MF
cat META-INF/MANIFEST.MF
```

On Windows:

```cmd
jar xf target\shade-demo-1.0.0.jar META-INF\MANIFEST.MF
type META-INF\MANIFEST.MF
```

Expected entry:

```text
Main-Class: com.example.App
```

---

# Why Main-Class Matters

Without `Main-Class`, Java does not know which class to start.

Error example:

```text
no main manifest attribute
```

The Shade Plugin transformer fixes this by adding:

```text
Main-Class: com.example.App
```

---

# Step 10: Compare Normal and Uber JAR

Normal JAR:

```text
Contains only project classes
Needs dependency classpath
```

Uber JAR:

```text
Contains project classes + dependency classes
Can run directly
```

This is useful for deployment and demonstration.

---

# Optional: Customize Final JAR Name

Add this inside `<build>`:

```xml
<finalName>my-executable-app</finalName>
```

Output becomes:

```text
target/my-executable-app.jar
```

Command:

```bash
java -jar target/my-executable-app.jar
```

---

# Optional: Package Relocation

Shade Plugin can relocate packages.

Example:

```xml
<relocations>
    <relocation>
        <pattern>com.google.gson</pattern>
        <shadedPattern>com.example.shaded.gson</shadedPattern>
    </relocation>
</relocations>
```

This helps avoid dependency conflicts.

---

# Why Package Relocation Is Useful

Relocation prevents conflict when:

- Two libraries use different versions of the same package.
- Application and dependency have overlapping classes.
- Multiple frameworks bring incompatible versions.

It renames dependency packages inside the shaded JAR.

---

# Maven Lifecycle Connection

The Shade Plugin is bound to:

```text
package phase
```

So when we run:

```bash
mvn package
```

the plugin automatically executes.

This is an example of Maven plugin execution.

---

# CI/CD Pipeline Example

```yaml
name: Build Uber JAR

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
      - uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build Uber JAR
        run: mvn clean package
```

---

# Artifact Upload Example

```yaml
      - name: Upload JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: shade-demo-jar
          path: target/shade-demo-1.0.0.jar
```

This saves the generated uber JAR as a CI artifact.

---

# Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `no main manifest attribute` | Main class missing | Add ManifestResourceTransformer |
| `ClassNotFoundException` | Dependencies not packaged | Use Shade Plugin |
| Large JAR size | All dependencies included | Review dependency need |
| Duplicate files warning | Same resource in dependencies | Use transformers |
| Wrong main class | Incorrect package/class | Fix `mainClass` value |

---

# Advantages of Uber JAR

| Advantage | Explanation |
|---|---|
| Easy distribution | One file only |
| Simple execution | `java -jar app.jar` |
| No classpath setup | Dependencies included |
| CI/CD friendly | Easy artifact upload |
| Useful for demos | Simple deployment |

---

# Limitations of Uber JAR

| Limitation | Explanation |
|---|---|
| Larger file size | Dependencies increase size |
| Possible duplicate classes | Needs careful handling |
| Harder dependency visibility | Libraries are bundled |
| Not always ideal for containers | Docker layers may be less efficient |

---

# When to Use Shade Plugin

Use Shade Plugin when:

- You need a single executable JAR.
- The app is a CLI tool.
- Distribution should be simple.
- Users should not manage dependencies.
- Deployment environment is minimal.

---

# When Not to Use Shade Plugin

Avoid Shade Plugin when:

- Application server manages dependencies.
- WAR deployment is required.
- Container layering optimization is important.
- Dependencies must remain separate for licensing or scanning.
- Build size must remain very small.

---

# Practical Verification Checklist

Students should verify:

- Gson dependency is added.
- Shade Plugin is configured.
- Main class is correctly declared.
- `mvn clean package` succeeds.
- Shaded JAR is generated in `target`.
- `java -jar` runs successfully.
- JAR content includes dependency packages.
- Manifest contains `Main-Class`.

---

# Final Answer Summary

The Maven Shade Plugin creates an uber JAR that contains both application code and required dependencies.

This solves the problem of missing external libraries during execution.

By binding the Shade Plugin to the `package` phase and setting the `Main-Class`, the team can build and run the application using:

```bash
mvn clean package
java -jar target/shade-demo-1.0.0.jar
```

---

# Short Exam-Style Answer

The Maven Shade Plugin is used to create an executable uber JAR, also known as a fat JAR. An uber JAR contains the application classes along with all required dependency classes, so users do not need to manually configure external libraries. To implement it, add dependencies in `pom.xml`, configure the Maven Shade Plugin under the `package` phase, and define the main class using `ManifestResourceTransformer`. After running `mvn clean package`, the final JAR can be executed using `java -jar`.

---

# Student Activity

Perform the following:

1. Create a Maven project.
2. Add a Java class using Gson.
3. Add Gson dependency in `pom.xml`.
4. Configure Maven Shade Plugin.
5. Add main class in manifest transformer.
6. Run `mvn clean package`.
7. Execute the shaded JAR.
8. List JAR contents using `jar tf`.
9. Check manifest file.
10. Explain difference between normal JAR and uber JAR.

---

# End of Scenario 9

## Topic Covered

**Maven Shade Plugin and executable uber JAR creation**

---
