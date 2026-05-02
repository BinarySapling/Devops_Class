# Master Viva & Technical Placement Q&A Bank: Maven Build Automation
## Target Audience: University Practical Exams & Software Engineering Recruitment Rounds

This comprehensive compilation consolidates high-yield questions, conceptual checkups, and advanced scenario-based questions from all five study days of the Maven syllabus. Use this guide for last-minute revision before practical exams or technical placement interviews.

---

## Category 1: Easy & Conceptual (Foundations & Directories)

### Q1: What is Maven, and what is the Project Object Model (POM)?
* **Answer**: Apache Maven is a declarative build automation and dependency management tool for Java applications. The **Project Object Model (POM)** is defined by the **`pom.xml`** file in the project root directory. It contains all metadata about the project, including its coordinates, dependencies, build configurations, and plugin settings.

### Q2: What are the three core coordinates that uniquely identify any Maven artifact?
* **Answer**: They are known as **GAV coordinates**:
  1. **`groupId`**: Identifies the organization or project namespace (e.g. `com.company.module`).
  2. **`artifactId`**: The unique name of the specific project module/binary (e.g. `payment-service`).
  3. **`version`**: The version number of the artifact (e.g. `1.2.0-SNAPSHOT` or `2.0.0`).

### Q3: Explain the concept of "Convention over Configuration".
* **Answer**: Maven enforces standard conventions (pre-defined rules) for project configurations, such as the standard directory tree structure (production code in `src/main/java`, test code in `src/test/java`, build outputs in `target/`). Adhering to these conventions eliminates the need to write complex configurations to specify where files reside, as Maven locates them automatically out-of-the-box.

### Q4: Describe the standard directory structure of a Maven project.
* **Answer**:
  * `src/main/java/`: Production Java source code files.
  * `src/main/resources/`: Configuration properties, logging settings, and static assets.
  * `src/test/java/`: Unit and integration test source code.
  * `src/test/resources/`: Configurations and data files used exclusively during testing.
  * `target/`: Transient directory where compiled classes and final packages (JAR/WAR) are written (git-ignored).
  * `pom.xml`: Declarative XML build configuration in the root folder.

### Q5: Where does Maven store downloaded dependencies locally? How can you configure this?
* **Answer**: Maven caches all downloaded dependencies in the local repository directory, which defaults to **`~/.m2/repository/`** on the developer's home machine. This path can be customized by editing the `<localRepository>` tag inside the global `~/.m2/settings.xml` configuration file.

---

## Category 2: Medium & Scenario-Based (Lifecycles, Scopes & Inheritance)

### Q6: What are the three built-in build lifecycles in Maven?
* **Answer**:
  1. **`default`**: Manages application compilation, testing, packaging, and deployments.
  2. **`clean`**: Manages deleting build outputs and clearing temporary compile states.
  3. **`site`**: Manages generating HTML documentation based on project metadata.

### Q7: List the core phases of the default build lifecycle in order.
* **Answer**:
  1. `validate`
  2. `compile`
  3. `test`
  4. `package`
  5. `verify`
  6. `install`
  7. `deploy`

### Q8: If you execute the command `mvn install`, will Maven run unit tests? Explain why.
* **Answer**: Yes. In the Default lifecycle, `test` is an upstream phase that precedes `install`. When you trigger a build phase in Maven, it automatically executes **every preceding phase** in that lifecycle. Therefore, `mvn install` automatically runs `validate`, `compile`, `test`, `package`, and `verify` first.

### Q9: Contrast the `compile`, `provided`, and `test` dependency scopes.
* **Answer**:
  * **`compile`**: The library is required in all classpaths (compilation, testing, execution) and is bundled directly into the final packaged binary.
  * **`provided`**: Required during compilation and testing but is **not** packaged because the target web container provides it at runtime (e.g. `servlet-api`).
  * **`test`**: Required exclusively for test compilation and execution (e.g. `junit`). It is completely omitted from production packages.

### Q10: What is Project Inheritance (Parent POM)? How does it differ from Project Aggregation (Modules)?
* **Answer**:
  * **Inheritance (Parent POM)**: Relies on the `<parent>` tag. It allows a child project to inherit configurations, properties, dependencies, and plugins from a parent POM to **reuse configurations**.
  * **Aggregation (Multi-Module)**: Relies on the `<modules>` tag defined in a root POM. It aggregates multiple independent projects to **orchestrate builds** together with a single command.

---

## Category 3: Hard & Enterprise (Conflict Resolution, Wrapper & Docker)

### Q11: Explain the "Nearest Wins" rule in Maven dependency conflict resolution.
* **Answer**: When different versions of the same dependency are imported transitively through separate paths, Maven resolves the conflict by selecting the version that is closest to the project root in the dependency tree (i.e. has the shortest depth path).

### Q12: What is the tie-breaker rule if two conflicting dependency versions are at the exact same depth in the tree?
* **Answer**: Maven uses the **"First Declared"** rule. It selects the dependency version whose path is declared first in the `pom.xml` dependencies block.

### Q13: What is the purpose of `<dependencyManagement>` inside a Parent POM? How does it differ from `<dependencies>`?
* **Answer**:
  * **`<dependencyManagement>`**: Declares allowed dependency coordinates (version, scope, exclusions) but **does not download them or add them to any classpath**. Sub-modules must explicitly declare the dependency in their local `<dependencies>` block, omitting the version tag, to automatically inherit the managed version. This centralizes version control.
  * **`<dependencies>`**: Instantly downloads and adds the declared libraries to the classpath, automatically inheriting them in all sub-modules.

### Q14: How does the Maven Surefire Plugin differ from the Maven Failsafe Plugin?
* **Answer**:
  * **Surefire Plugin**: Runs unit tests during the **`test`** phase. If a unit test fails, the build is aborted immediately.
  * **Failsafe Plugin**: Runs integration tests during the **`integration-test`** and **`verify`** phases. It is designed to allow the build to proceed past failures so cleanup steps can run, reporting integration test failures at the end of the `verify` phase.

### Q15: What is an Uber-JAR (or Fat-JAR)? How does the Maven Shade Plugin compile it?
* **Answer**: An Uber-JAR is a single, self-contained executable JAR archive containing your compiled project classes *and* the extracted class files of all transitively declared dependencies.
  * The **Shade Plugin** extracts all class files from dependency JARs, merges them into a single archive, relocates conflicting packages if configured, and updates the manifest metadata to specify the application's main entry point class.

### Q16: What is Class Relocation in the Shade Plugin? Why is it crucial in enterprise setups?
* **Answer**: Class relocation modifies the package path of a dependency at the bytecode level during packaging (e.g. renaming `org.gson` to `shaded.org.gson`). This resolves runtime classpath conflicts, allowing your application to bundle a specific version of a library without conflicting with other versions present on the target application server or execution cluster.

### Q17: What is the Maven Wrapper (`mvnw`)? Why is it essential for CI/CD pipelines?
* **Answer**: The Maven Wrapper provides portable, version-aligned builds. It consists of shell scripts and configuration files committed to Git that automatically download and run the declared Maven version if it is missing on the host machine. This ensures consistent builds across different developer machines and clean CI/CD runners without pre-installing Maven.

### Q18: Describe how a Multi-stage Dockerfile optimizes Java container builds.
* **Answer**:
  * **Stage 1 (Builder)**: Uses a heavy base image containing Maven and JDK to copy the project files, download dependencies, and compile the executable Uber-JAR.
  * **Stage 2 (Runtime)**: Uses a minimal base image containing only the JRE, copying *only* the compiled Uber-JAR from Stage 1 and discarding all source files, compilers, and local download caches. This reduces the final container size and minimizes security vulnerabilities.

### Q19: How do you optimize dependency caching inside Docker builds?
* **Answer**: By copying only the `pom.xml` file first and executing **`mvn dependency:go-offline`** before copying the source code directory. This downloads all dependencies and caches them in a dedicated Docker layer, preventing Docker from re-downloading dependencies on subsequent builds unless the `pom.xml` changes.

### Q20: What is a Bill of Materials (BOM) in Maven?
* **Answer**: A BOM is a special POM file that defines a curated, compatible set of dependency versions inside its `<dependencyManagement>` block. Other projects can import this BOM using the `<scope>import</scope>` and `<type>pom</type>` coordinates, allowing them to use any dependency in the set without specifying version tags, ensuring mutual compatibility out-of-the-box.
