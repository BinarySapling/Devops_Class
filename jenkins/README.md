# Jenkins CI/CD Study Notes: Complete 5-Day Syllabus
## Level: University Practical Exam + Technical Placement Preparation

Welcome to the comprehensive self-study directory and repository for **Jenkins CI/CD**. This curriculum has been engineered to help students master enterprise continuous integration, Maven compilation builds, containerized dynamic agent pipelines, Shared Libraries, and remote server deployment workflows.

---

## 📅 Study Program Map

Navigate through the theoretical notes, architectural flowcharts, production-grade Groovy scripts, and placement checkups for each day:

| Day | Date | Focus Areas | Navigation Links |
| :--- | :--- | :--- | :--- |
| **Day 1** | **11 May 2026** | Jenkins distributed architecture (Master/Agent model), Outbound SSH vs Inbound JNLP connections, Installation, Plugin management, and Role-Based Strategy security. | [Day 1 Notes](./01-11-may-2026.md) |
| **Day 2** | **12 May 2026** | Pipelines-as-Code. Freestyle vs Pipeline jobs, Declarative wrapper (`pipeline`) vs Scripted wrapper (`node`), environment scopes, runtime parameters, and Multi-Branch scanning. | [Day 2 Notes](./02-12-may-2026.md) |
| **Day 3** | **13 May 2026** | Jenkins and Maven integration, tools binding, automated unit testing, JUnit XML report parsing, JaCoCo coverage, post build actions, and archiving binaries. | [Day 3 Notes](./03-13-may-2026.md) |
| **Day 4** | **14 May 2026** | Containerized Pipelines. Docker inside Jenkins (DooD vs DinD), stage-level containers, authenticating with registries (Docker Hub & GHCR), GitHub Webhook automation, and Backup & Restore. | [Day 4 Notes](./04-14-may-2026.md) |
| **Day 5** | **15 May 2026** | Continuous Delivery. Global Shared Pipelines Libraries (src, vars, resources), dynamic agent scaling (Kubernetes pods), SSH/SFTP agents, and cloud staging deployments. | [Day 5 Notes](./05-15-may-2026.md) |

---

## 🎯 Master Revision & Consolidated Resources

* **💬 [Consolidated Viva Q&A Bank](./viva-questions.md)**: Over 40 university exam viva and technical placement questions categorized by difficulty, complete with standard professional answers.
* **🧪 [Hands-on Practical Lab Manual](./practical-lab-manual.md)**: A complete series of 6 step-by-step practical laboratory experiments to build, run, and debug Jenkinsfiles and Groovy libraries locally and in staging VM environments.

---

## 🚀 Key Course Objectives

By completing this curriculum, you will master:
1. **Distributed System Orchestration**: Building Master/Agent grids using secure connection paths (SSH/JNLP).
2. **Pipeline-as-Code (Groovy)**: Writing robust Declarative and Scripted Jenkinsfiles with strict parameter validation and post conditions.
3. **Enterprise Compiler Tooling**: Linking build automation tools (Maven, JDK) and generating test reports (JUnit) dynamically.
4. **Virtual Sandboxes (Docker)**: Running pipeline stages inside temporary containers, publishing OCI images to Docker Hub/GHCR, and mount configurations (DooD vs DinD).
5. **Reusability at Scale**: Writing Centralized Shared Libraries (`vars/`) to eliminate configuration drift across enterprise repositories.
