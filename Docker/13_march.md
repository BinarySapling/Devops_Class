# Docker Internals: Namespaces, cgroups, and Images

## Real World Analogy Behind Namespaces
It is like separate cabins in a train:
*   Same engine (kernel)
*   Different passengers
*   No visibility between cabins

This means each container works independently but uses the same system.

## Control Groups (cgroups)
cgroups are Linux kernel features used to control and limit resource usage of containers.

**They ensure:**
*   No container uses all resources
*   Fair sharing of CPU, memory, and I/O

> **Note:** Namespaces provide isolation, while cgroups provide control.

### Need of cgroups
Without cgroups:
*   One container can consume all CPU or RAM.
*   Other containers and the host system may crash.

### Types of Resources Controlled
*   **CPU:** Limit usage and assign shares.
*   **Memory:** Set max limit, kill if exceeded.
*   **Disk I/O:** Control read/write.
*   **Network:** Controlled indirectly.

### Real World Analogy (cgroups)
Like electricity meters in flats:
*   Each flat has a limit.
*   One person cannot use all power.

---

## Tasks / Scenarios

### Task 1
**Scenario:** If a faulty container crashes the system due to high memory.
*   **Missing feature:** cgroups
*   **Kernel mechanism:** cgroups
*   **Are namespaces alone enough?:** No (they only provide isolation, no control).

### Task 2
**Scenario:** 3 containers (A → CPU intensive, B → web server, C → database).
*   **CPU control:** cgroups
*   **Process isolation:** PID namespace
*   **Same port usage:** Network namespace

---

## Container Images
An **Image** is a read-only blueprint of a container.

**Includes:**
*   Base OS
*   Application code
*   Libraries
*   Runtime
*   Config

> Images are **immutable** (cannot change once created).
> *Example:* `python:3.11-slim`

### Layering
Images are built in layers:
`OS` → `runtime` → `application` → `dependencies`

### Image vs Container
*   **Image:** Read-only blueprint.
*   **Container:** Writable running instance.

---

## Image Registry
A central place to store and manage images.

**Uses:**
*   Share images.
*   Same environment everywhere.
*   Works with CI/CD.

### Why Registry Needed?
**Without it:**
*   Build again and again
*   No version control
*   Inconsistency

**With it:**
*   Build once, use everywhere
*   Faster deployment
*   Easy rollback

### Types
*   **Public:** Open access.
*   **Private:** Secure and restricted.

### Examples
*   Amazon Elastic Container Registry (ECR)
*   Azure Container Registry (ACR)

### Image Naming Format
`<registry>/<namespace>/<image>:<tag>`
*Example:* `docker.io/aastha/webapp:v1`

### Push and Pull
*   **Push:** Upload image.
*   **Pull:** Download image.
*(Only new layers are transferred)*

### CI/CD Flow
`Code` → `Build Image` → `Push` → `Deploy`

### Comparison
*   **Public Registry:** Less secure
*   **Private Registry:** More secure and controlled
