# Practical Laboratory Manual: DevOps Engineering (Maven Build Automation)
## Target Audience: University Practical Exams, Lab Record Submissions, & DevOps Engineers

This laboratory manual provides step-by-step practical experiments for mastering Maven Build Automation. Each experiment is structured with a clear **Objective**, **Prerequisites**, **Source Code (XML/Dockerfile/Bash)**, **Execution Steps**, **Verification Methods**, and **Troubleshooting Guidelines** to serve as a complete lab record or study guide.

---

## 🧪 Table of Laboratory Experiments

1. **Lab 1**: Initializing a Standard Maven Project & Managing Core POM Coordinates
2. **Lab 2**: Configuring Parent POM Inheritance & Module Aggregation
3. **Lab 3**: Resolving Transitive Dependency Version Conflicts & Exclusions
4. **Lab 4**: Packaging an Executable Self-Contained Uber-JAR using the Shade Plugin
5. **Lab 5**: Bootstrapping Portable CI/CD Pipeline Builds with the Maven Wrapper (`mvnw`)
6. **Lab 6**: Compiling a Lightweight JRE Docker Container via a Multi-Stage Build

---

## 📘 Experiment 1: Initializing a Standard Maven Project & Managing Core POM Coordinates

### 1. Objective
To understand Maven's Standard Directory Layout, configure a basic Project Object Model (`pom.xml`) with coordinate configurations (GAV), compile a Java source file, and inspect compiler outputs.

### 2. Project Directory Setup
Manually create the standardized Maven directory structure:
```bash
mkdir -p src/main/java/com/company/app
mkdir -p src/main/resources
mkdir -p src/test/java/com/company/app
mkdir -p src/test/resources
```

### 3. Workflow Configuration (`pom.xml`)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company.app</groupId>
    <artifactId>hello-maven</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
</project>
```

### 4. Application Code (`src/main/java/com/company/app/App.java`)
```java
package com.company.app;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello, Maven! Core build automation successful.");
    }
}
```

### 5. Execution & Verification Steps
1. Navigate to the project root directory containing `pom.xml`.
2. Execute the compiler phase:
   ```bash
   mvn compile
   ```
3. Inspect your filesystem. Verify that a directory named `target/` was created at the root level, containing compiled bytecode classes:
   `target/classes/com/company/app/App.class`.
4. Run the application from the compiled classes using Maven:
   ```bash
   mvn exec:java -Dexec.mainClass="com.company.app.App"
   ```
5. Verify the console output displays: `"Hello, Maven! Core build automation successful."`.

---

## 📘 Experiment 2: Configuring Parent POM Inheritance & Module Aggregation

### 1. Objective
To construct a multi-module enterprise project hierarchy, configure property and dependency propagation using Parent POM inheritance, and aggregate compilation builds from a single root command.

### 2. Directory Layout Setup
```bash
mkdir -p multi-module-demo/core-module/src/main/java/com/company/core
```

### 3. Parent POM (`multi-module-demo/pom.xml`)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company.enterprise</groupId>
    <artifactId>parent-orchestrator</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <shared-log-version>2.20.0</shared-log-version>
    </properties>

    <!-- Project Aggregation: Lists child sub-directories to compile together -->
    <modules>
        <module>core-module</module>
    </modules>

    <!-- Centralized Dependency Management -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-core</artifactId>
                <version>${shared-log-version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 4. Child POM (`multi-module-demo/core-module/pom.xml`)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.company.enterprise</groupId>
        <artifactId>parent-orchestrator</artifactId>
        <version>1.0.0</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>core-module</artifactId>

    <dependencies>
        <!-- Child inherits G and V coordinates, compiler versions, and imports log4j without specifying a version -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 5. Execution & Verification Steps
1. Navigate to the root directory `multi-module-demo/`.
2. Run the packaging phase:
   ```bash
   mvn package
   ```
3. Verify in the execution logs that Maven builds **both** projects, outputting:
   * `Reactor Build Order: parent-orchestrator -> core-module`
   * `parent-orchestrator ... SUCCESS`
   * `core-module ........ SUCCESS`
4. Confirm that `core-module/target/core-module-1.0.0.jar` was created successfully.

---

## 📘 Experiment 3: Resolving Transitive Dependency Version Conflicts & Exclusions

### 1. Objective
To map and analyze transitive dependencies in a project using the Maven dependency tree, and use the `<exclusions>` tag to resolve library version conflicts.

### 2. Workflow Configuration (`pom.xml`)
```xml
<dependencies>
    <!-- Declares ActiveMQ dependency, which transitively imports old logging frameworks -->
    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-client</artifactId>
        <version>5.18.3</version>
        <exclusions>
            <!-- Excludes transitively imported legacy slf4j dependency to resolve conflicts -->
            <exclusion>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- Explicitly import the desired slf4j-api version coordinates -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version>
    </dependency>
</dependencies>
```

### 3. Execution & Verification Steps
1. Create a `pom.xml` containing the dependencies above *without* the `<exclusions>` block.
2. Execute the dependency tree generator:
   ```bash
   mvn dependency:tree
   ```
3. Locate the duplicate `slf4j-api` version branches. Note the conflict.
4. Add the `<exclusions>` block to your `pom.xml` as shown.
5. Run `mvn dependency:tree` again.
6. Verify in the output tree log that the legacy transitive version has been removed, leaving only the desired version `2.0.9` bound to the project classpath.

---

## 📘 Experiment 4: Packaging an Executable Self-Contained Uber-JAR using the Shade Plugin

### 1. Objective
To configure the `maven-shade-plugin` to compile all production source files and transitively declared dependency libraries into a single, self-contained, executable Uber-JAR archive.

### 2. Workflow Configuration (`pom.xml`)
```xml
<project>
    <!-- GAV Coordinates -->
    <groupId>com.company.app</groupId>
    <artifactId>uber-jar-demo</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <!-- Gson Dependency (Will be packaged inside the Uber-JAR) -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.10.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.5.1</version>
                <executions>
                    <execution>
                        <!-- Bind automatically to package lifecycle phase -->
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <!-- Defines the Main Class manifest attribute -->
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.company.app.App</mainClass>
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

### 3. Application Code (`src/main/java/com/company/app/App.java`)
```java
package com.company.app;

import com.google.gson.Gson;
import java.util.HashMap;
import java.util.Map;

public class App {
    public static void main(String[] args) {
        Gson gson = new Gson();
        Map<String, String> data = new HashMap<>();
        data.put("status", "SUCCESS");
        data.put("message", "Uber-JAR compiled and executed successfully!");
        
        System.out.println(gson.toJson(data));
    }
}
```

### 4. Execution & Verification Steps
1. Run the packaging command:
   ```bash
   mvn clean package
   ```
2. Navigate to the `target/` directory. You will find two JAR files:
   * `original-uber-jar-demo-1.0.jar` (thin JAR, only containing your classes).
   * `uber-jar-demo-1.0.jar` (Uber-JAR containing GSON class files).
3. Execute the Uber-JAR using Java:
   ```bash
   java -jar target/uber-jar-demo-1.0.jar
   ```
4. Verify the console prints the JSON payload successfully, proving that GSON dependencies are packaged inside the JAR.

---

## 📘 Experiment 5: Bootstrapping Portable CI/CD Pipeline Builds with the Maven Wrapper

### 1. Objective
To generate the portable Maven Wrapper files, commit them to version control, and compile your project without pre-installing a Maven engine on the host system.

### 2. Execution & Generation Steps
1. Navigate to your project directory.
2. Generate the wrapper configurations pointing to a specific target version:
   ```bash
   mvn wrapper:wrapper -Dmaven=3.9.6
   ```
3. Inspect the project filesystem to verify that the following files have been generated:
   * `mvnw` (Unix shell script).
   * `mvnw.cmd` (Windows batch script).
   * `.mvn/wrapper/maven-wrapper.properties` (specifies the download URL).
   * `.mvn/wrapper/maven-wrapper.jar` (bootstrapping executor).
4. Stage and commit all wrapper assets to Git:
   ```bash
   git add mvnw mvnw.cmd .mvn/
   git commit -m "docs: add Maven wrapper scripts"
   ```

### 3. Verification Steps
1. (Optional) Temporarily uninstall Maven from your path.
2. Execute the wrapper compile script:
   * Linux/macOS: `./mvnw clean package`
   * Windows: `.\mvnw.cmd clean package`
3. Verify in the output logs that the script automatically downloads Maven 3.9.6, caches it inside your home folder, and uses it to package the project successfully.

---

## 📘 Experiment 6: Compiling a Lightweight JRE Docker Container via a Multi-Stage Build

### 1. Objective
To write an efficient Multi-Stage `Dockerfile` that compile your Java application inside a builder container, leverages layer caching, copies only the compiled JAR to a minimal Alpine JRE runtime container, and pushes the final image to a registry.

### 2. Workflow Configuration (`Dockerfile` in project root)
```dockerfile
# ==========================================
# STAGE 1: Compiler Build Container
# ==========================================
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /app

# Copy the pom.xml first to leverage Docker layer caching
COPY pom.xml .

# Download dependencies offline
RUN mvn dependency:go-offline -B

# Copy source directories
COPY src ./src

# Compile and package the Uber-JAR
RUN mvn package -DskipTests

# ==========================================
# STAGE 2: Production JRE Container
# ==========================================
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Copy only the compiled Uber-JAR from Stage 1
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

# Configure a secure non-root user
USER 10001:10001

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 3. Execution & Verification Steps
1. Navigate to the project root directory containing the `Dockerfile`.
2. Build the Docker image:
   ```bash
   docker build -t maven-app:latest .
   ```
3. Run the container:
   ```bash
   docker run -d -p 8080:8080 --name my-running-java-app maven-app:latest
   ```
4. Verify the container is running and outputs logs successfully:
   ```bash
   docker logs my-running-java-app
   ```
5. Run `docker images` and verify the final image size is minimal (around 150MB), containing only the JRE and your compiled JAR file.

---

## 🛠️ General Troubleshooting Guidelines

### 1. POM XML Validation Syntax Errors
* **Symptom**: `FATAL error: ... non-parseable POM`.
* **Solution**: Ensure your XML tags are balanced and correctly nested. Verify that elements inside `pom.xml` use valid schemas.

### 2. Maven Wrapper Permission Denied
* **Symptom**: `./mvnw: Permission denied` on Linux/macOS.
* **Solution**: Git may have stripped the execution permission bit from the script. Restore it by running:
  ```bash
  chmod +x mvnw
  ```

### 3. Maven Dependency Offline Cache Fails
* **Symptom**: `mvn dependency:go-offline` fails due to unresolved dependencies.
* **Solution**: Ensure your machine or container has active internet access to reach the Maven Central Repository. If using custom repositories, ensure the corresponding `<repositories>` blocks are declared in `pom.xml`.
