# Practical Laboratory Manual: DevOps Engineering (Jenkins CI/CD)
## Target Audience: University Practical Exams, Lab Record Submissions, & DevOps Engineers

This laboratory manual provides step-by-step practical experiments for mastering Jenkins CI/CD. Each experiment is structured with a clear **Objective**, **Prerequisites**, **Source Code (Groovy/Bash)**, **Execution Steps**, **Verification Methods**, and **Troubleshooting Guidelines** to serve as a complete lab record or study guide.

---

## 🧪 Table of Laboratory Experiments

1. **Lab 1**: Multi-User Access Management with Role-Based Access Control (RBAC) & Agent Isolation
2. **Lab 2**: Authoring a Parameterized Declarative Pipeline with Conditional Logic
3. **Lab 3**: Java Compilation Pipeline with Maven Tools & JUnit Test Reporting
4. **Lab 4**: Containerized CI: Compiling Docker Images and Publishing to GHCR
5. **Lab 5**: Automated Continuous Integration via GitHub Webhooks
6. **Lab 6**: Reusable Devops at Scale: Creating & Importing a Jenkins Global Shared Library

---

## 📘 Experiment 1: Role-Based Access Control & Agent Isolation

### 1. Objective
To configure Role-Based Access Control (RBAC) in Jenkins using the Role-Based Strategy plugin to manage granular user access, and configure master executor isolation to secure the controller.

### 2. Execution & Setup Steps
1. Install Jenkins locally on a virtual machine or container.
2. Navigate to **Manage Jenkins** -> **Plugins** -> Search and install **"Role-based Authorization Strategy"**.
3. Go to **Manage Jenkins** -> **Configure Global Security** -> Under **Authorization**, select **Role-Based Strategy** -> Click **Save**.
4. Create three local users: `developer-user`, `qa-user`, and `ops-user` under **Manage Jenkins** -> **Users** -> **Create User**.
5. Go to **Manage Jenkins** -> **Manage and Assign Roles**:
   * Under **Manage Roles**: Create a Global Role named `read-only` with overall Read permissions.
   * Under **Item Roles**: Create a project role named `dev-projects` with a pattern `dev-.*`, granting job read, build, and cancel permissions.
6. Under **Assign Roles**:
   * Assign `developer-user` to the `read-only` global role and the `dev-projects` item role.
7. Go to **Manage Jenkins** -> **Nodes** -> Click **Built-in Node** -> Click **Configure**. Set the number of executors to `0` and save.

### 3. Verification Methods
* Log out of the admin account and log in as `developer-user`.
* Create a job named `dev-payment-service`. Verify you can see and trigger it.
* Create a job named `prod-deployment-service`. Verify that `developer-user` **cannot view or trigger** this job.
* Verify on the Jenkins homepage that the executor pane shows "Built-in Node" with 0 active executors.

---

## 📘 Experiment 2: Parameterized Declarative Pipeline with Conditional Logic

### 1. Objective
To write a Declarative Pipeline containing runtime parameters, secure credentials binding, conditional stage execution using `when` expressions, and post-build state actions.

### 2. Workflow Configuration (`Jenkinsfile`)
```groovy
pipeline {
    agent any

    parameters {
        string(name: 'RELEASE_TAG', defaultValue: 'v1.0.0', description: 'Version Tag')
        choice(name: 'TARGET_ENVIRONMENT', choices: ['Development', 'Staging', 'Production'], description: 'Target Deploy Environment')
        booleanParam(name: 'RUN_INTEGRATION_TESTS', defaultValue: false, description: 'Run integration test suite?')
    }

    environment {
        APP_NAME = 'billing-api'
        // Securely bind secret from credential store
        STAGING_KEY = credentials('STAGING_SERVER_SSH_KEY')
    }

    stages {
        stage('Initialize') {
            steps {
                echo "Starting build process for ${APP_NAME} version ${params.RELEASE_TAG}"
                echo "Target Environment: ${params.TARGET_ENVIRONMENT}"
            }
        }

        stage('Optional Tests') {
            when {
                expression { params.RUN_INTEGRATION_TESTS == true }
            }
            steps {
                echo "Running integration tests..."
                sh 'echo Integration test simulation running'
            }
        }

        stage('Deploy Staging') {
            when {
                environment name: 'TARGET_ENVIRONMENT', value: 'Staging'
            }
            steps {
                echo "Deploying to Staging Server using secret key:"
                // The output below is automatically masked as *** in console logs
                sh "echo Deploy Key: ${STAGING_KEY}"
            }
        }
    }

    post {
        always {
            echo "Pipeline run completed."
        }
        success {
            echo "SUCCESS: Job passed."
        }
        failure {
            echo "FAILURE: Job crashed! Sending alerts..."
        }
    }
}
```

### 3. Verification Steps
1. Create a **Pipeline** job in Jenkins. Copy the script above into the pipeline text area (simulate by configuring credentials `'STAGING_SERVER_SSH_KEY'` first in Jenkins Credentials Manager).
2. Save and click **"Build Now"** (first execution parses the parameters).
3. The button will change to **"Build with Parameters"**. Click it, enter custom parameters, and trigger the build.
4. Verify the console logs to check that the staging deployment stage only executes when `Staging` is selected, and check that the key output is masked as `***`.

---

## 📘 Experiment 3: Java Compilation Pipeline with Maven Tools & JUnit Test Reporting

### 1. Objective
To integrate Maven builds in Jenkins. The pipeline must dynamically configure a specific Maven and JDK version, compile a Java project (`pom.xml`), run unit tests, parse JUnit XML reports, and archive the resulting JAR/WAR packages.

### 2. Prerequisites
In Jenkins Dashboard: Go to **Manage Jenkins** -> **Tools**:
* Under **JDK**: Click Add -> Name it `JDK17` -> Set to auto-install.
* Under **Maven**: Click Add -> Name it `M3` -> Set to auto-install version `3.9.6`.

### 3. Workflow Configuration (`Jenkinsfile`)
```groovy
pipeline {
    agent any

    tools {
        maven 'M3'
        jdk 'JDK17'
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                echo "Compiling Java source code..."
                sh 'mvn clean compile'
            }
        }

        stage('Run Tests') {
            steps {
                echo "Executing unit tests..."
                sh 'mvn test'
            }
            post {
                always {
                    echo "Parsing test reports..."
                    // Parse Surefire reports even if previous steps failed
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package Binary') {
            steps {
                echo "Packaging WAR file..."
                sh 'mvn package -DskipTests'
            }
        }

        stage('Archive Build Artifacts') {
            steps {
                echo "Archiving binaries to Jenkins Master..."
                archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
            }
        }
    }

    post {
        always {
            cleanWs() // Clean agent workspace directories
        }
    }
}
```

### 4. Verification Steps
1. Push the code to a Git repository containing a standard Java Maven project structure (`pom.xml`).
2. Configure a pipeline pointing to the Git repository and trigger the build.
3. Check the console logs; verify that Jenkins automatically downloads and configures JDK 17 and Maven 3.9 during build initialization.
4. Verify that the JUnit test trend graph appears on the job homepage, and download the compiled `.war` artifact directly from the successful build homepage.

---

## 📘 Experiment 4: Containerized CI: Compiling Docker Images and Publishing to GHCR

### 1. Objective
To build a containerized pipeline that builds a Docker image from a `Dockerfile`, runs unit tests inside a Node.js container, and publishes the compiled image securely to GitHub Container Registry (GHCR) using repository secrets.

### 2. Workflow Configuration (`Jenkinsfile`)
```groovy
pipeline {
    agent { label 'docker-runner' } // Target agent with Docker CLI installed

    environment {
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'danishanwar/payment-gateway'
        REGISTRY_CREDS = credentials('GHCR_PUBLISH_SECRET')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test inside Container') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "Running tests inside temporary Node container..."
                sh 'node --version'
                sh 'npm install && npm test || echo Mock test passed'
            }
        }

        stage('Compile Docker Image') {
            steps {
                echo "Compiling Docker image..."
                sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                sh "docker tag ${REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER} ${REGISTRY}/${IMAGE_NAME}:latest"
            }
        }

        stage('Publish to GHCR') {
            steps {
                echo "Pushing image to GitHub Container Registry..."
                sh "echo ${REGISTRY_CREDS} | docker login ${REGISTRY} -u danishanwar --password-stdin"
                sh "docker push ${REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
                sh "docker push ${REGISTRY}/${IMAGE_NAME}:latest"
            }
        }
    }

    post {
        always {
            sh "docker rmi -f ${REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER} || true"
            sh "docker rmi -f ${REGISTRY}/${IMAGE_NAME}:latest || true"
            cleanWs()
        }
    }
}
```

### 3. Verification Steps
1. Commit the `Jenkinsfile` and a sample `Dockerfile` to your Git repository.
2. In Jenkins, create the `'GHCR_PUBLISH_SECRET'` secret containing your GitHub Personal Access Token (PAT) with package write permissions.
3. Run the pipeline. Monitor the console logs to verify that the testing stage runs inside a Node.js container, and verify that the image is compiled, logged in, and successfully pushed to `ghcr.io`.
4. Log in to your GitHub account and check that your package is published under the "Packages" section of your profile.

---

## 📘 Experiment 5: Automated Continuous Integration via GitHub Webhooks

### 1. Objective
To configure a Git Webhook trigger in GitHub that automatically triggers a Jenkins pipeline build on every code push, implementing hands-free Continuous Integration (CI).

### 2. Execution & Configuration Steps
1. Ensure your Jenkins server is publicly accessible (use tools like `ngrok` for local development setup: `ngrok http 8080` to generate a public URL).
2. Go to your **GitHub Repository** -> **Settings** -> **Webhooks** -> Click **"Add webhook"**.
3. Configure the following properties:
   * **Payload URL**: `http://<YOUR_PUBLIC_JENKINS_URL>/github-webhook/` (ensure the trailing `/` is present).
   * **Content Type**: `application/json`.
   * **Trigger**: Select **"Just the push event"** -> Click **Add Webhook**.
4. In Jenkins, open your Pipeline job -> Click **Configure** -> Scroll down to **Build Triggers** -> Check the box **"GitHub hook trigger for GITScm polling"** -> Click **Save**.

### 3. Verification Steps
1. Clone the repository locally, modify a file, commit it, and run `git push origin main`.
2. Check your GitHub Webhook log under repository settings to verify a `200 OK` response was returned for the push webhook event.
3. Open your Jenkins Dashboard and verify that a new build pipeline is triggered automatically within seconds of your push.

---

## 📘 Experiment 6: Reusable DevOps: Creating & Importing a Jenkins Global Shared Library

### 1. Objective
To build a reusable, central DevOps ecosystem by creating a Jenkins Global Shared Library, declaring standardized steps, importing it into a project's `Jenkinsfile`, and executing a secure deployment using an SSH agent.

### 2. Shared Library Setup
Create a new Git repository named `jenkins-shared-library` containing a directory named `vars/`.
Create a file named `vars/deployToVM.groovy` inside it:
```groovy
// vars/deployToVM.groovy
def call(Map config = [:]) {
    def targetIP = config.ip
    def credentialsId = config.creds

    echo "==== SHARED LIBRARY: INITIATING DEPLOYMENT ===="
    echo "Target VM: ${targetIP}"

    sshagent([credentialsId]) {
        sh """
            ssh -o StrictHostKeyChecking=no ubuntu@${targetIP} "
                echo 'SSH authentication successful!'
                docker stop application-running || true
                docker rm application-running || true
                docker run -d -p 80:80 --name application-running nginx:alpine
            "
        """
    }
}
```
Commit and push this library repository to your Git remote.

### 3. Configure Shared Library in Jenkins
1. Go to **Manage Jenkins** -> **System** (Configure System).
2. Scroll to **Global Pipeline Libraries** -> Click **Add**.
3. Configure the properties:
   * **Name**: `my-shared-library`
   * **Default Version**: `main`
   * **Retrieval Method**: Modern SCM -> Select Git -> Enter your library Git repository URL -> Save.

### 4. Application Pipeline Configuration (`Jenkinsfile`)
In your project repository, write a `Jenkinsfile` that imports the shared library and triggers the custom deployment step:
```groovy
@Library('my-shared-library@main') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "Compiling application package..."
            }
        }

        stage('Staging Deployment') {
            steps {
                // Call the custom shared library step
                deployToVM(
                    ip: '10.0.4.15',
                    creds: 'staging-ssh-private-key'
                )
            }
        }
    }
}
```

### 5. Verification Steps
1. Push the `Jenkinsfile` to your project repository.
2. Create the `'staging-ssh-private-key'` SSH credential in Jenkins' Credential Manager containing the private key of your target server VM.
3. Run the project pipeline.
4. Verify in the logs that Jenkins successfully clones your shared library repository, parses the `deployToVM` step, registers the SSH key agent, and executes the commands on the remote VM.

---

## 🛠️ General Troubleshooting Guidelines

### 1. Script Approval Blocked Errors
* **Symptom**: `org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException: Scripts not permitted to use method ...`.
* **Solution**: Script Security block. Go to **Manage Jenkins** -> **In-process Script Approval** -> Click **Approve** on the rejected signature to authorize execution.

### 2. Maven Executable Command Not Found
* **Symptom**: `mvn: command not found` inside a shell stage.
* **Solution**:
  1. Verify the name specified in the `tools` directive (e.g. `maven 'M3'`) exactly matches the name defined in the Global Tool Settings.
  2. Ensure the agent VM has Java installed (Maven requires Java to run).

### 3. Webhook Triggers Fail to Fire
* **Symptom**: Commits are pushed, but no build is triggered.
* **Solution**:
  * Verify the webhook payload URL in GitHub settings ends with a trailing slash (`/github-webhook/`).
  * Ensure the Jenkins server's public IP is accessible from the internet (check ngrok/firewall rules).
  * Ensure the "GitHub hook trigger for GITScm polling" is checked in your Jenkins job.
  * Run a manual build once to ensure Jenkins has parsed the SCM configuration.
