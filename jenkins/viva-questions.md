# Master Viva & Technical Placement Q&A Bank: Jenkins CI/CD
## Target Audience: University Practical Exams & Software Engineering Recruitment Rounds

This comprehensive compilation consolidates high-yield questions, conceptual checkups, and advanced scenario-based questions from all five study days of the Jenkins syllabus. Use this guide for last-minute revision before practical exams or technical placement interviews.

---

## Category 1: Easy & Conceptual (Core Architecture & Security)

### Q1: What is Jenkins, and what is its primary role in DevOps?
* **Answer**: Jenkins is an open-source, Java-based automation server designed to facilitate Continuous Integration and Continuous Deployment (CI/CD). It automates the software development lifecycle, allowing teams to automatically check out code, run compilers, execute tests, package binaries, build containers, and deploy to servers.

### Q2: What is the significance of the `$JENKINS_HOME` directory?
* **Answer**: `$JENKINS_HOME` is the primary filesystem directory where Jenkins stores all its system data. This includes global system configuration XML files (`config.xml`), credentials configurations, installed plugin binaries (`plugins/` directory), pipeline job configurations and their build history (`jobs/` directory), and private security encryption keys (`secrets/` directory).

### Q3: Explain the Jenkins Master/Agent distributed architecture.
* **Answer**:
  * **Jenkins Master (Controller)**: Handles orchestrating tasks, hosting the web dashboard interface, managing global security configurations, parsing pipeline syntax, storing credentials, and scheduling build workloads.
  * **Jenkins Agent (Worker)**: A lightweight Java executor process (`agent.jar`) running on separate VM hosts. It connects to the Master, listens for execution instructions, runs the compilation/test steps locally, and streams log outputs back to the Master in real-time.

### Q4: How do Agents connect to the Jenkins Master?
* **Answer**:
  * **Outbound SSH Connection**: The Master connects to the Agent over SSH (port 22) using stored private credentials, pushes the `agent.jar` file, and initiates the process. This is the preferred topology for Linux nodes.
  * **Inbound JNLP Connection**: The Agent machine connects inward to the Master's JNLP TCP port (usually 50000) using a secure secret token. This is preferred for Windows agents or nodes behind isolated private firewalls.

### Q5: What are the default ports utilized by a standard Jenkins installation?
* **Answer**: The Jenkins Web UI runs on HTTP port **`8080`** by default. Inbound JNLP Agent connections typically use TCP port **`50000`** (or a dynamically allocated port).

### Q6: How do you recover access to Jenkins if an administrative user is locked out?
* **Answer**: Open `$JENKINS_HOME/config.xml` on the host machine, locate the `<useSecurity>true</useSecurity>` tag, change the value to `false`, and restart the Jenkins service. This completely disables security access controls, allowing you to re-configure administrative users.

---

## Category 2: Medium & Scenario-Based (Pipelines, Maven & Webhooks)

### Q7: Contrast Freestyle Jobs with Jenkins Pipelines.
* **Answer**:
  * **Freestyle Jobs**: Configured entirely through interactive forms on the Jenkins Web UI. They are hard to version control, review, template, or recover if deleted.
  * **Jenkins Pipelines**: Declared as code in a **`Jenkinsfile`** (written in Groovy) and committed directly to the project's Git repository. This enables version control, code review, auditing, and allows pipelines to resume automatically if the controller crashes.

### Q8: Compare Declarative and Scripted pipeline syntax.
* **Answer**:
  * **Declarative Pipeline**: Uses a structured, readable DSL wrapped in a `pipeline { ... }` block. It has strict formatting rules, predefined execution scopes, and automated error handling via the `post` block. It is the modern industry standard.
  * **Scripted Pipeline**: Uses raw Groovy script wrapped in a `node { ... }` block. It provides maximum flexibility, has no structure limits, and handles errors using standard `try-catch-finally` blocks. It has a steeper learning curve.

### Q9: What is the purpose of the `tools` directive block in a Declarative Pipeline?
* **Answer**: It automatically configures pre-defined build compilers and tools (such as specific Maven, JDK, or Gradle versions defined in the Global Tool Configuration) and binds them to the pipeline's execution environment path (`PATH`) dynamically before the stages run.

### Q10: How does Jenkins integrate with Maven for Java applications?
* **Answer**:
  1. Define a Maven installation in **Global Tool Configuration** (e.g., named `'M3'`).
  2. In the `Jenkinsfile`, declare `tools { maven 'M3' }`.
  3. Execute Maven goals inside shell steps, e.g., `sh 'mvn clean package'`.
  4. Parse the output test reports using the `junit '**/target/surefire-reports/*.xml'` step.

### Q11: Explain the difference between `archiveArtifacts` and Jenkins build caching.
* **Answer**:
  * **`archiveArtifacts`**: Copies successfully compiled build packages (e.g. `.jar`, `.war` binaries) from the transient agent workspace to the persistent Master storage so they can be downloaded from the Web UI.
  * **Build Caching**: Saves package manager dependencies (e.g. Maven `.m2` repository or npm `node_modules`) to avoid re-downloading them in subsequent runs, accelerating build speeds.

### Q12: How does a Jenkins Multi-Branch Pipeline work?
* **Answer**: It automatically scans a Git repository to discover active branches. For every branch it finds containing a `Jenkinsfile`, it automatically provisions a separate sub-project and runs a build pipeline. If a branch is deleted or merged, Jenkins automatically archives or removes its sub-job.

### Q13: What does the "UNSTABLE" build status mean? How does it differ from "FAILED"?
* **Answer**:
  * **UNSTABLE (Yellow)**: The build steps and compilation completed successfully, but a post-build step reported a failure (typically failing **unit tests** parsed by the `junit` plugin).
  * **FAILED (Red)**: The pipeline execution crashed due to syntax errors, compiler failure, missing tool dependencies, or a shell command exiting with a non-zero code.

---

## Category 3: Hard & Enterprise (Docker, Shared Libraries & Disaster Recovery)

### Q14: Explain the differences between Docker-out-of-Docker (DooD) and Docker-in-Docker (DinD).
* **Answer**:
  * **DooD (Docker-out-of-Docker)**: The Jenkins agent container mounts the host’s Docker socket `/var/run/docker.sock`. Command-line executions are processed by the host VM's Docker daemon.
    * *Pros*: Faster builds because it shares the host's image caches.
    * *Cons*: High security risk; the agent container gains administrative control over the host host.
  * **DinD (Docker-in-Docker)**: The agent runs a completely isolated, nested instance of the Docker daemon inside its own container boundaries.
    * *Pros*: Complete sandbox isolation.
    * *Cons*: Slower builds (cannot share host image caches easily) and requires running the agent container in `--privileged` mode.

### Q15: How do you configure a GitHub webhook to trigger a Jenkins job on every push?
* **Answer**:
  1. In GitHub repository settings, add a webhook with the URL `http://<jenkins-ip>:8080/github-webhook/` (trailing slash is mandatory).
  2. Set Content-Type to `application/json`.
  3. In your Jenkins job, check the option **"GitHub hook trigger for GITScm polling"**.
  4. Upon code push, GitHub sends a POST request with the commit metadata, which triggers the matching pipeline.

### Q16: What is a Jenkins Global Shared Library? Why is it used?
* **Answer**: A shared library is a centralized Git repository containing reusable Groovy helper functions and pipeline steps. It is imported at the top of a `Jenkinsfile` using `@Library('my-library@version') _`. It is used to standardize pipeline structures across hundreds of repositories, eliminating copy-paste configuration drift and simplifying maintenance.

### Q17: Describe the directory structure of a Jenkins Shared Library.
* **Answer**:
  * `src/`: Standard Object-Oriented Groovy classes containing helper code.
  * `vars/`: Global variable files whose filenames define custom pipeline steps callable directly in a `Jenkinsfile` (e.g. `vars/buildApp.groovy` enables the `buildApp()` step).
  * `resources/`: Non-Groovy helper files (JSON config templates, SQL schemas) read by pipeline steps.

### Q18: What is a Dynamic Container Agent in Jenkins? What are its benefits?
* **Answer**: A dynamic container agent is a temporary container spun up on-demand (typically on a Kubernetes cluster) when a build is scheduled. The container executes the assigned stages and is terminated immediately upon completion.
  * *Benefits*: Eliminates the need to maintain static agent VMs, scales infinitely based on concurrent build demands, provides absolute resource isolation, and saves infrastructure costs.

### Q19: Explain a disaster recovery backup strategy for a Jenkins Master. What should you exclude?
* **Answer**: Write an automated script (or use the ThinBackup plugin) to create a compressed tar archive of `$JENKINS_HOME` daily.
  * **Include**: Global and job-level configuration XML files (`config.xml`, `jobs/**/*.xml`), the `plugins/` folder, decryption keys under `secrets/`, and the `users/` database.
  * **Exclude**: The `workspace/` directory (large, transient files easily re-cloned from Git) and optionally historical `builds/**/log` files to keep the backup archive compact.

### Q20: Explain the `sshagent` block and why it is critical for secure staging deployments.
* **Answer**: `sshagent` is a wrapper step provided by the SSH Agent Plugin. It securely loads SSH private keys from Jenkins' credential store and registers them with a temporary SSH key agent in the background for the duration of the block. This allows pipeline steps to securely run `scp` or `ssh` commands to connect and deploy to remote VMs without writing key material or passwords to temporary workspace files or exposing them in shell history.
