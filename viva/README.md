# DevOps Viva Preparation & Comprehensive Question Bank

> **Subject:** LYCS DevOps Sem 7  
> **Topics Covered:** Git, Jenkins, Docker, Ansible, Kubernetes, Monitoring (ELK, Prometheus & Grafana)  
> **Format:** Detailed Explanations, Comparison Tables, Code Snippets, Direct Viva Answers, and 10 One-line Quick Revision.

---

## Table of Contents
- [Part 1: Git, Jenkins and Docker](#part-1-git-jenkins-and-docker)
  - [1. Difference between Git Merge and Git Rebase](#1-difference-between-git-merge-and-git-rebase)
  - [2. What is the use of Jenkinsfile?](#2-what-is-the-use-of-jenkinsfile)
  - [3. Difference between Docker Image and Container](#3-difference-between-docker-image-and-container)
  - [4. What is the use of `docker-compose.yml`?](#4-what-is-the-use-of-docker-composeyml)
  - [5. Explain CI vs CD](#5-explain-ci-vs-cd)
- [Part 2: Ansible, Kubernetes and Monitoring](#part-2-ansible-kubernetes-and-monitoring)
  - [6. Difference between Ansible Ad-hoc Command and Playbook](#6-difference-between-ansible-ad-hoc-command-and-playbook)
  - [7. What is a Pod in Kubernetes? How is it different from a Container?](#7-what-is-a-pod-in-kubernetes-how-is-it-different-from-a-container)
  - [8. What is the use of Deployment vs Service in Kubernetes?](#8-what-is-the-use-of-deployment-vs-service-in-kubernetes)
  - [9. What are the 3 components of ELK Stack and their roles?](#9-what-are-the-3-components-of-elk-stack-and-their-roles)
  - [10. Difference between Prometheus and Grafana](#10-difference-between-prometheus-and-grafana)
- [Quick Revision – 10 One-Line Answers](#quick-revision--10-one-line-answers)
- [Last-Minute Viva Tips](#last-minute-viva-tips)

---

# Part 1: Git, Jenkins and Docker

## 1. Difference between Git Merge and Git Rebase

Both Git Merge and Git Rebase are used to combine changes from different branches.

### Git Merge
- Combines changes from two branches.
- Creates a merge commit when necessary.
- Preserves the original branch history and chronological commit timeline.
- Does not rewrite existing commits.

### Git Rebase
- Moves the commits of one branch on top of another branch.
- Creates a cleaner, linear commit history.
- Rewrites commit history by creating brand new commits with new hashes.
- Makes the project history easier to follow and read.

### Comparison Table

| Feature / Aspect | Git Merge | Git Rebase |
| :--- | :--- | :--- |
| **History Structure** | Combines branches while preserving their full branched history. | Moves commits onto another branch to create a clean, linear history. |
| **Merge Commit** | May create a dedicated merge commit (e.g. 3-way merge). | Never creates a merge commit; applies commits linearly. |
| **Commit Rewriting** | Does not rewrite existing commits. | Rewrites the rebased branch's commit history with new commit IDs. |
| **Best Used For** | Public/shared branches where history preservation is required. | Local feature branches before opening a PR/merging into main. |

> **Example:** If you have a `main` branch and a `feature` branch, **Merge** combines their histories by joining them at a merge commit. **Rebase** replays the `feature` branch commits on top of the latest tip of the `main` branch.

> **💡 Viva Answer:**  
> *"Git Merge combines branches while preserving their original history and may create a merge commit, whereas Git Rebase moves commits onto another branch to create a clean, linear history by rewriting commits with new hashes."*

---

## 2. What is the use of Jenkinsfile?

A **Jenkinsfile** is a text file that defines the entire CI/CD pipeline of a project using Pipeline-as-Code. It tells Jenkins how to build, test, and deploy an application.

### Key Uses:
- **Defines Pipeline Stages:** Clearly outlines stages such as `Checkout`, `Build`, `Test`, `Package`, and `Deploy`.
- **Automates Tasks:** Automates software development, compilation, testing, and distribution workflows.
- **Pipeline as Code:** Maintains pipeline configuration versioned alongside application source code in Git.
- **Repeatability & Traceability:** Makes the build and deployment process repeatable, auditable, and identical across environments.

### Example Jenkinsfile (Declarative Syntax)
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building application'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing application'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application'
            }
        }
    }
}
```

> **💡 Viva Answer:**  
> *"A Jenkinsfile is used to define and automate a CI/CD pipeline as code. It contains different stages like Build, Test, and Deploy, which Jenkins executes automatically whenever triggered."*

> **📌 Possible Follow-Up:**  
> Jenkinsfile uses a Groovy-based Domain-Specific Language (DSL) and supports both **Declarative** (structured, recommended) and **Scripted** (imperative, programmatic) syntax.

---

## 3. Difference between Docker Image and Container

A **Docker Image** is an immutable, read-only template containing application code, system libraries, and dependencies. A **Docker Container** is a running, isolated execution instance of that image.

### Comparison Table

| Feature / Aspect | Docker Image | Docker Container |
| :--- | :--- | :--- |
| **Nature** | It is a blueprint/template. | It is a running instance of an image. |
| **Layer Mutability** | Read-only. | Has a thin top writable (container) layer. |
| **Function** | Used to package and create containers. | Used to run applications and processes. |
| **Execution** | Does not run by itself (passive file). | Executes the application process (active runtime). |
| **Storage Lifecycle** | Persists in image registries / local disk. | Ephemeral; can be started, stopped, moved, or deleted. |

> **Analogy:** Think of a **Docker Image as a recipe** (or class in OOP), and a **Docker Container as the actual food prepared** (or an instantiated object).

> **💡 Viva Answer:**  
> *"A Docker Image is a read-only template containing the application code and its dependencies, whereas a Docker Container is a running instance of that image with an added writable layer."*

---

## 4. What is the use of `docker-compose.yml`?

**Docker Compose** is a tool used to define, configure, and run multi-container Docker applications. The `docker-compose.yml` file contains the complete service configuration required by the application stack.

### Key Uses:
- **Runs Multiple Containers Together:** Orchestrates multi-tier apps (e.g. web, API, database) from a single YAML file.
- **Service Definition:** Specifies images, build contexts, environment variables, and ports.
- **Automated Networking:** Automatically establishes a shared internal network so containers resolve each other by service name.
- **Volume Management:** Mounts host volumes or named volumes for persistent data storage.
- **Single-Command Control:** Starts and stops the entire application stack using simple CLI commands.

### Example `docker-compose.yml`
```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"

  database:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: password
```

### Essential Commands:
- `docker compose up -d` — Builds, creates, and starts all defined services in the background.
- `docker compose down` — Stops and removes containers, networks, and volumes created by Compose.

> **💡 Viva Answer:**  
> *"The `docker-compose.yml` file is used to define and manage multi-container Docker applications together using a single configuration file, configuring their services, networking, and storage volumes."*

---

## 5. Explain CI vs CD

**CI/CD** stands for **Continuous Integration** and **Continuous Delivery / Continuous Deployment**.

### CI – Continuous Integration
- Developers frequently commit code changes into a shared central repository (e.g. GitHub).
- Every commit triggers automated builds and automated test suites.
- Identifies integration bugs and syntax errors early in development.
- Enhances code quality and reduces integration debt.

### CD – Continuous Delivery vs. Continuous Deployment
- **Continuous Delivery:** Automatically builds, tests, and stages the application for release. The final push to production requires a manual approval step.
- **Continuous Deployment:** Every code change that passes all pipeline test stages is automatically deployed directly into production without manual human intervention.

### Comparison Table

| Feature / Aspect | Continuous Integration (CI) | Continuous Delivery / Deployment (CD) |
| :--- | :--- | :--- |
| **Primary Focus** | Integrating, compiling, and testing developer code. | Releasing, staging, and deploying tested artifacts. |
| **Automation Scope** | Automates code compilation and automated test runs. | Automates packaging, infrastructure provisioning, and deployment. |
| **Core Value** | Detects bugs early and prevents broken builds. | Delivers features and bug fixes to end users rapidly and reliably. |

> **💡 Viva Answer:**  
> *"CI automatically integrates, builds, and tests code changes whenever a developer commits, while CD automates the packaging and deployment of the tested application to staging or production environments."*

---

# Part 2: Ansible, Kubernetes and Monitoring

## 6. Difference between Ansible Ad-hoc Command and Playbook

Ansible is an open-source IT automation engine used for configuration management, server provisioning, and task execution.

### Ansible Ad-hoc Command
- Used for quick, one-time, single-task execution.
- Runs directly from the terminal without creating a YAML file.
- Fast and suitable for health checks, reboots, or user creation.
- **Example:**
  ```bash
  ansible all -m ping
  ```
  *(Tests connectivity to all managed hosts in the inventory).*

### Ansible Playbook
- Used for complex, multi-task, repeatable configuration management.
- Written in structured **YAML** format.
- Tasks execute in a strict, sequential order.
- Reusable, version-controllable, and idempotent.
- **Example:**
  ```yaml
  - name: Install Apache
    hosts: all
    become: yes
    tasks:
      - name: Install Apache package
        apt:
          name: apache2
          state: present
  ```

### Comparison Table

| Feature / Aspect | Ad-hoc Command | Ansible Playbook |
| :--- | :--- | :--- |
| **Complexity** | Simple, single-operation tasks. | Complex, multi-step orchestration and configuration. |
| **Syntax Format** | Executed directly on the command line. | Written in structured YAML files (`.yml`). |
| **Reusability** | One-time execution (hard to reuse). | Reusable, repeatable, and maintainable in Git. |
| **Execution Target** | Executes a single module (e.g. `-m ping`). | Executes a series of tasks using multiple modules. |

> **💡 Viva Answer:**  
> *"Ansible Ad-hoc commands are quick terminal commands used for one-time operations like pinging hosts, whereas Playbooks are structured YAML files used to automate complex, multi-step server configurations in a repeatable manner."*

---

## 7. What is a Pod in Kubernetes? How is it different from a Container?

A **Pod** is the smallest and most basic deployable compute unit in Kubernetes. It encapsulates one or more containers that share storage, network resources, and a specification for how to run the containers.

### Key Features of a Pod:
- Can contain one main application container (most common) or tightly coupled helper containers (sidecar pattern).
- Containers within a Pod share the same **Network Namespace** (same IP address and port space; communicate via `localhost`).
- Containers can share shared storage **Volumes**.
- Every Pod receives a unique internal cluster IP address.

### Comparison Table

| Feature / Aspect | Pod | Container |
| :--- | :--- | :--- |
| **Definition** | Smallest deployable unit managed by Kubernetes. | Standard unit that packages and runs application code. |
| **Composition** | Can host one or multiple co-located containers. | Single application process running in isolated user space. |
| **Management** | Managed and scheduled by the Kubernetes control plane. | Managed by a container runtime (e.g. containerd, Docker). |
| **Networking** | Containers inside a Pod share the Pod IP and network namespace. | Isolated network namespace per container by default. |

> **Analogy / Example:** A web server container and a log-forwarding container running together in the same Pod can share local disk storage and communicate over `localhost`.

> **💡 Viva Answer:**  
> *"A Pod is the smallest deployable unit in Kubernetes that encapsulates one or more containers. Containers inside the same Pod share the same IP address, network namespace, and storage volumes."*

> **📌 Possible Follow-Up:**  
> *Can a Pod have multiple containers?* Yes, this is known as a multi-container Pod (such as the Sidecar, Adapter, or Ambassador patterns), though the 1-container-per-pod pattern is the standard.

---

## 8. What is the use of Deployment vs Service in Kubernetes?

Both **Deployment** and **Service** are core Kubernetes resources, but they solve two different problems: **Lifecycle Management** vs. **Networking**.

### Deployment (Workload Controller)
- Declares and manages application Pods.
- Maintains the desired number of Pod replicas (e.g. `replicas: 3`).
- Automatically heals crashed Pods by recreating them.
- Supports zero-downtime **Rolling Updates** and instant rollbacks.

### Service (Networking Abstraction)
- Provides a stable, permanent IP address and DNS endpoint for dynamic Pods.
- Dispatches and load-balances incoming traffic across all matching Pods using label selectors (`app: myapp`).
- Enables decoupled microservice communication within and outside the cluster.

### Comparison Table

| Feature / Aspect | Deployment | Service |
| :--- | :--- | :--- |
| **Primary Role** | Manages Pod lifecycles, replicas, and rolling updates. | Provides stable networking, DNS, and load balancing for Pods. |
| **Target** | Focuses on application instances and versioning. | Focuses on network endpoints and traffic routing. |
| **Pod IP Problem** | Pods are ephemeral; their IPs change when recreated. | Solves the dynamic IP problem by providing one unchanging virtual IP. |
| **Key Controller** | ReplicaSet controller. | Endpoints / Kube-Proxy controller. |

> **Example:** If an application runs across 3 Pods, the **Deployment** guarantees that all 3 Pods remain healthy and scales them. The **Service** exposes a single virtual IP (e.g. port 80) and load-balances user requests across those 3 Pods.

> **💡 Viva Answer:**  
> *"A Deployment manages the lifecycle, scaling, and rolling updates of application Pods, whereas a Service provides a stable network endpoint, DNS name, and load balancer to access those Pods."*

---

## 9. What are the 3 components of ELK Stack and their roles?

The **ELK Stack** (now often called the Elastic Stack) is a powerful suite of three open-source tools designed for centralized log collection, processing, searching, and visualization.

### The 3 Core Components:
1. **Elasticsearch (Storage & Search Engine):**  
   A distributed, JSON-based search and analytics engine. It indexes, stores, and enables near-real-time full-text search across massive volumes of log data.
2. **Logstash (Data Processing Pipeline):**  
   A server-side data collection engine. It ingests logs from diverse inputs, parses, filters, enriches, and transforms them into structured JSON, then forwards them to Elasticsearch.
3. **Kibana (Visualization & Dashboarding):**  
   A web UI frontend that sits on top of Elasticsearch. It allows engineers to search logs, create visual charts, histograms, heatmaps, and build interactive operational monitoring dashboards.

### End-to-End Workflow:
```
[Application / Server Logs]
            |
            v
   [ 1. Logstash ]  (Collects, parses, and enriches raw logs)
            |
            v
 [ 2. Elasticsearch ] (Indexes and stores structured JSON documents)
            |
            v
    [ 3. Kibana ]  (Visualizes dashboards, queries, and error metrics)
```

> **💡 Viva Answer:**  
> *"The ELK Stack consists of Elasticsearch, Logstash, and Kibana. Logstash collects and transforms log data, Elasticsearch indexes and stores it for high-speed search, and Kibana visualizes the logs via interactive dashboards."*

---

## 10. Difference between Prometheus and Grafana

**Prometheus** and **Grafana** are the industry standard open-source monitoring and observability stack.

### Prometheus (Metrics Aggregation & Alerting)
- Operates primarily on a **Pull Model** (periodically scrapes `/metrics` HTTP endpoints).
- Stores data in its own dedicated, optimized **Time-Series Database (TSDB)**.
- Features **PromQL** (Prometheus Query Language) for mathematical metric aggregation.
- Contains **Alertmanager** to trigger notifications (email, Slack, PagerDuty).

### Grafana (Visualization & Dashboarding Engine)
- Dedicated visualization and dashboard engine.
- Does **not** store time-series metrics natively; queries existing data sources (Prometheus, InfluxDB, CloudWatch, Elasticsearch).
- Renders customizable panels, graphs, gauges, and alerts.

### Comparison Table

| Feature / Aspect | Prometheus | Grafana |
| :--- | :--- | :--- |
| **Primary Function** | Scrapes, collects, stores metrics, and evaluates alerts. | Visualizes metrics through customizable dashboards. |
| **Data Storage** | Built-in high-performance Time-Series Database (TSDB). | No built-in TSDB; connects to external databases. |
| **Data Collection** | Pulls (scrapes) metrics from monitored targets. | Queries Prometheus or other data sources via API. |
| **User Interface** | Basic query evaluation and expression graph browser. | Rich, interactive, production-grade dashboards. |

> **Example:** To monitor server CPU, memory, and disk usage, **Prometheus** scrapes and stores metrics from the `node_exporter` agent every 15 seconds, while **Grafana** displays these metrics as real-time line charts and visual gauges on a unified dashboard.

> **💡 Viva Answer:**  
> *"Prometheus is used to scrape, store, and alert on time-series monitoring metrics, whereas Grafana is used to visualize those metrics through customizable charts, graphs, and dashboards."*

---

# Quick Revision – 10 One-Line Answers

| No. | Question / Topic | One-Line Summary Answer |
| :---: | :--- | :--- |
| **1** | **Git Merge vs. Rebase** | Merge combines branches while preserving history; Rebase creates a linear history by replaying commits. |
| **2** | **Jenkinsfile** | A text file that defines and automates the CI/CD pipeline stages as code. |
| **3** | **Docker Image vs. Container** | An Image is a read-only template; a Container is an active, runnable instance of that image. |
| **4** | **`docker-compose.yml`** | A YAML file that defines, configures, and runs multi-container Docker applications together. |
| **5** | **CI vs. CD** | CI automatically integrates and tests code; CD automates delivery or deployment to environments. |
| **6** | **Ansible Ad-hoc vs. Playbook** | Ad-hoc commands execute quick single tasks; Playbooks automate complex, repeatable workflows in YAML. |
| **7** | **Pod vs. Container** | A Pod is the smallest Kubernetes deployable unit and can host one or more co-located containers. |
| **8** | **Deployment vs. Service** | A Deployment manages and scales Pod instances; a Service provides a stable IP and load balances traffic. |
| **9** | **ELK Stack Components** | Logstash processes logs, Elasticsearch indexes/stores them, and Kibana visualizes them on dashboards. |
| **10** | **Prometheus vs. Grafana** | Prometheus scrapes and stores time-series metrics; Grafana visualizes those metrics in dashboards. |

---

# Last-Minute Viva Tips

1. **Master the Paired Comparisons:**  
   Examiners frequently ask for contrast between paired tools:
   - *Merge vs. Rebase*
   - *Image vs. Container*
   - *CI vs. CD*
   - *Ad-hoc Command vs. Playbook*
   - *Pod vs. Container*
   - *Deployment vs. Service*
   - *Prometheus vs. Grafana*

2. **Structure Your Answer in 3 Steps:**
   - **Step 1:** Give a concise 1-sentence definition of both terms.
   - **Step 2:** Highlight the main difference (e.g. *template vs. running instance*).
   - **Step 3:** Give a practical, real-world example (e.g. *recipe vs. dish*).

3. **Avoid Pure Rote Memorization:**  
   Focus on *why* a DevOps tool exists (e.g. why do we need a Kubernetes Service? Because Pod IPs are dynamic and change on every crash).

4. **When Stuck:**  
   State the core problem the tool solves first, then describe its basic command or configuration file.
