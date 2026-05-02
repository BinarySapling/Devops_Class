# Maven Build Automation Study Notes: Complete 5-Day Syllabus
## Level: University Practical Exam + Technical Placement Preparation

Welcome to the comprehensive self-study directory and repository for **Maven Build Automation**. This curriculum has been designed to help students master declarative dependency management, transitive dependency trees, compiler plugins, custom goal phase executions, executable packaging (Uber-JARs), and Docker container integrations.

---

## 📅 Study Program Map

Navigate through the theoretical notes, compiler tree diagrams, production-grade XML configurations, and placement checkups for each day:

| Day | Date | Focus Areas | Navigation Links |
| :--- | :--- | :--- | :--- |
| **Day 1** | **27 April 2026** | Why build tools exist, classpath issues, Project Object Model (`pom.xml`) base structure, GAV coordinates, packaging indicators, and Maven's Standard Directory layout. | [Day 1 Notes](./01-27-apr-2026.md) |
| **Day 2** | **28 April 2026** | Built-in Lifecycles (Clean, Site, Default), core execution phases (`compile`, `test`, `package`, `install`, `deploy`), and Parent POM inheritance. | [Day 2 Notes](./02-28-apr-2026.md) |
| **Day 3** | **29 April 2026** | Classpath Scopes (compile, provided, runtime, test, import), transitive dependency trees, version conflict resolution algorithms (nearest-wins), exclusions, and Bill of Materials (BOM) integration. | [Day 3 Notes](./03-29-apr-2026.md) |
| **Day 4** | **30 April 2026** | Extensibility via Plugins. Compiler plugin, Surefire plugin, Shade plugin configurations (creating executable Uber-JARs), and the Maven Wrapper (`mvnw`). | [Day 4 Notes](./04-30-apr-2026.md) |
| **Day 5** | **1 May 2026** | Maven and Docker integration. Multi-stage Dockerfiles compiling and packaging inside containers, `docker-maven-plugin` configuration, and pushing to registries. | [Day 5 Notes](./05-01-may-2026.md) |

---

## 🎯 Master Revision & Consolidated Resources

* **💬 [Consolidated Viva Q&A Bank](./viva-questions.md)**: Over 40 university exam viva and technical placement questions categorized by difficulty, complete with standard professional answers.
* **🧪 [Hands-on Practical Lab Manual](./practical-lab-manual.md)**: A complete series of 6 step-by-step practical laboratory experiments to configure POM files, resolve conflicts, package Uber-JARs, configure the wrapper, and compile multi-stage Docker containers.

---

## 🚀 Key Course Objectives

By completing this curriculum, you will master:
1. **Declarative Build Management**: Restricting compilation behaviors via standardized XML configurations.
2. **Dynamic Classpath Scoping**: Separating compile, provided, and testing classpaths to reduce final production footprints.
3. **Advanced Dependency Engineering**: Resolving version conflicts (nearest-wins) and utilizing `<dependencyManagement>` to enforce version alignment.
4. **Self-Contained Executables**: Building single, portable, self-contained executable packages (Uber-JARs) containing all transitive library dependencies.
5. **CI/CD Build Portability**: Bootstrapping builds on clean CI/CD servers using the portable Maven Wrapper (`mvnw`).
6. **Containerized Build Isolation**: Utilizing multi-stage Docker builds to package lightweight, secure JRE applications.
