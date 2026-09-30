# DevOps Interview Prep 🚀

> **My practical DevOps Engineer interview-preparation repository — learn, practice, troubleshoot, document, and revise.**

This repository is a personal **quick-reference knowledge base** for DevOps engineering.

The primary goal is to keep important concepts, commands, configuration files, troubleshooting notes, examples, and interview material organized in one place so they can be revised quickly before an interview or used during hands-on practice.

---

## 📂 Complete Repository Structure

```text
Interview-Prep/
│
├── Docker/
│   └── Docker-Interview-prep-README.md
│
├── K8s/
│   ├── Minikube/
│   │   ├── 1 Minikube-Cluster-Set-up.md
│   │   ├── 2 Minikube-local-cluster-start-up.md
│   │   ├── 3 Introduction-to-yaml.md
│   │   ├── 4 Introduction-Pods.md
│   │   ├── 4.1 Local-K8s-API-Versions.txt
│   │   ├── 4.2 Sample-Nginx-Pod-Manifest.yaml
│   │   ├── 4.3 Basic-Kubectl-Commands.md
│   │   ├── 5.1 K8s-SelfHealing-High-Avaliability-nginx-replicaset.yaml
│   │   └── 5.1 K8s-SelfHealing-High-Avaliability.md
│   │
│   └── Kubernetes-Concepts.pdf
│
├── Linux/
│   └── Linux-Interview-prep-README.md
│
├── Terraform/
│   ├── Terraform-Folder-Structure.jpg
│   └── Terraform-Interview-prep-README.md
│
└── README.md
```

---

# 🎯 Repository Purpose

This repository is designed for **quick and practical DevOps Engineer interview preparation**.

Instead of keeping everything as theoretical notes, the repository focuses on:

- Core concepts
- Important commands
- YAML/configuration examples
- Hands-on exercises
- Troubleshooting
- Real command outputs
- Common interview questions
- Differences between similar concepts
- Quick revision notes

The idea is:

```text
Learn
  ↓
Understand
  ↓
Practice
  ↓
Break Things
  ↓
Troubleshoot
  ↓
Document
  ↓
Revise
  ↓
Interview
```

---

# 🧭 Current Topics

| Area | Status | Main Focus |
|---|---|---|
| 🐳 Docker | 📚 Preparing | Containers, images, Dockerfile, networking, volumes, troubleshooting |
| ☸️ Kubernetes | 🔥 Active | Minikube, YAML, Pods, kubectl, self-healing, high availability |
| 🐧 Linux | 📚 Preparing | Commands, processes, networking, permissions, troubleshooting |
| 🏗️ Terraform | 📚 Preparing | IaC, folder structure, resources, modules, state, workflows |

---

# 🐳 Docker

Location:

```text
Docker/
└── Docker-Interview-prep-README.md
```

The Docker section is intended to cover the practical knowledge required for DevOps interviews.

### Important areas

- Docker fundamentals
- Containers vs Images
- Docker architecture
- Dockerfile
- Docker image layers
- Container lifecycle
- Docker networking
- Volumes
- Environment variables
- Docker Compose
- Container logs
- Container troubleshooting
- Multi-stage builds
- Image optimization
- Common Docker commands
- Docker interview questions

---

# ☸️ Kubernetes

Location:

```text
K8s/
```

The Kubernetes section currently contains a **Minikube-based hands-on learning path** along with Kubernetes reference material.

## Minikube

```text
K8s/
└── Minikube/
```

### 1. Minikube Cluster Setup

```text
1 Minikube-Cluster-Set-up.md
```

Covers setting up a local Kubernetes environment for hands-on practice.

### 2. Starting the Local Cluster

```text
2 Minikube-local-cluster-start-up.md
```

Contains the steps required to bring the local cluster back up and running.

### 3. Introduction to YAML

```text
3 Introduction-to-yaml.md
```

Covers YAML fundamentals and syntax used for Kubernetes configuration.

### 4. Introduction to Pods

```text
4 Introduction-Pods.md
```

Introduces Kubernetes Pods and their basic concepts.

### 4.1 Local Kubernetes API Versions

```text
4.1 Local-K8s-API-Versions.txt
```

Reference information for Kubernetes API versions available in the local environment.

### 4.2 Sample Nginx Pod Manifest

```text
4.2 Sample-Nginx-Pod-Manifest.yaml
```

Hands-on Kubernetes Pod manifest example using Nginx.

### 4.3 Basic kubectl Commands

```text
4.3 Basic-Kubectl-Commands.md
```

Quick reference for frequently used `kubectl` commands.

Example:

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- bash
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
```

### 5.1 Self-Healing & High Availability

```text
5.1 K8s-SelfHealing-High-Avaliability.md
5.1 K8s-SelfHealing-High-Avaliability-nginx-replicaset.yaml
```

This topic combines theory with a practical ReplicaSet example to understand concepts such as:

- Desired state
- ReplicaSets
- Multiple Pod replicas
- Self-healing
- Pod replacement
- High availability fundamentals
- Labels and selectors

### Kubernetes Concepts Reference

```text
Kubernetes-Concepts.pdf
```

Additional Kubernetes reference material for deeper study.

---

# 🐧 Linux

Location:

```text
Linux/
└── Linux-Interview-prep-README.md
```

The Linux section is intended as a quick reference for the commands and troubleshooting skills expected from a DevOps Engineer.

### Important areas

- Linux fundamentals
- File system
- Files and directories
- Users and groups
- File permissions
- Processes
- Services
- Systemd
- Networking
- SSH
- Disk management
- CPU and memory troubleshooting
- Logs
- Package management
- Shell commands
- Bash scripting
- Environment variables
- Production troubleshooting

### Commands to master

```bash
ls
cd
pwd
find
grep
awk
sed
cat
less
head
tail
ps
top
df
du
free
chmod
chown
systemctl
journalctl
curl
wget
ssh
scp
ss
```

---

# 🏗️ Terraform

Location:

```text
Terraform/
├── Terraform-Folder-Structure.jpg
└── Terraform-Interview-prep-README.md
```

Terraform is maintained as the Infrastructure-as-Code preparation section.

### Important areas

- Infrastructure as Code
- Terraform architecture
- Providers
- Resources
- Variables
- Outputs
- Locals
- Data sources
- Terraform state
- Remote state
- State locking
- Modules
- Workspaces
- Dependencies
- Terraform lifecycle
- Troubleshooting
- Terraform interview questions

### Core workflow

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform destroy
```

The folder-structure image is kept as a visual reference for organizing Terraform projects.

---

# 🧠 How I Use This Repository

Every topic should ideally answer five questions:

### 1. What is it?

Understand the concept in simple terms.

### 2. Why do we need it?

Understand the problem it solves.

### 3. How does it work?

Understand the important internal flow/components.

### 4. How do I use it?

Practice with commands, configuration, and real examples.

### 5. How do I troubleshoot it?

Understand what to check when something goes wrong.

---

# 🔧 Troubleshooting Mindset

For DevOps interviews, knowing a command is not enough.

The important skill is being able to investigate a problem systematically.

```text
Problem
   ↓
What changed?
   ↓
What is the expected state?
   ↓
What is the actual state?
   ↓
Where is the failure?
   ↓
Check status
   ↓
Check configuration
   ↓
Check logs
   ↓
Check dependencies
   ↓
Reproduce the issue
   ↓
Apply the smallest safe fix
   ↓
Verify the result
```

Whenever I encounter an interesting error while practicing, the **problem + investigation + solution** should be documented in the relevant topic.

---

# ⚡ Quick Revision Philosophy

The notes should be useful when I have only a few minutes before an interview.

A good revision note should contain:

```text
Concept
   ↓
Key points
   ↓
Important commands
   ↓
Example
   ↓
Common mistakes
   ↓
Troubleshooting
   ↓
Interview questions
```

Avoid unnecessary textbook-style content.

Prefer:

- Short explanations
- Tables
- Command examples
- YAML/configuration examples
- Diagrams
- Practical scenarios
- Troubleshooting steps
- Interview questions

---

# 📋 Interview Preparation Checklist

## 🐳 Docker

- [ ] Docker architecture
- [ ] Images vs Containers
- [ ] Dockerfile
- [ ] Layers
- [ ] Volumes
- [ ] Networking
- [ ] Docker Compose
- [ ] Multi-stage builds
- [ ] Image optimization
- [ ] Container troubleshooting

## ☸️ Kubernetes

- [ ] Kubernetes architecture
- [ ] Control Plane
- [ ] Worker Nodes
- [ ] Pods
- [ ] ReplicaSets
- [ ] Deployments
- [ ] Services
- [ ] Labels & Selectors
- [ ] ConfigMaps
- [ ] Secrets
- [ ] Volumes
- [ ] Probes
- [ ] Requests & Limits
- [ ] Scheduling
- [ ] Self-healing
- [ ] High Availability
- [ ] Scaling
- [ ] Rolling Updates
- [ ] Rollbacks
- [ ] `kubectl`
- [ ] YAML manifests
- [ ] Troubleshooting

## 🐧 Linux

- [ ] Files & directories
- [ ] Permissions
- [ ] Users & groups
- [ ] Processes
- [ ] Services
- [ ] Systemd
- [ ] Networking
- [ ] SSH
- [ ] Disk troubleshooting
- [ ] CPU troubleshooting
- [ ] Memory troubleshooting
- [ ] Logs
- [ ] Bash scripting

## 🏗️ Terraform

- [ ] IaC fundamentals
- [ ] Providers
- [ ] Resources
- [ ] Variables
- [ ] Outputs
- [ ] Locals
- [ ] Data sources
- [ ] State
- [ ] Remote state
- [ ] State locking
- [ ] Modules
- [ ] Workspaces
- [ ] Dependencies
- [ ] Terraform workflow
- [ ] Troubleshooting

---

# 🗺️ Future Topics

As the preparation progresses, the repository can be expanded with:

- [ ] Git & GitHub
- [ ] CI/CD
- [ ] Jenkins
- [ ] GitHub Actions
- [ ] AWS
- [ ] Azure
- [ ] Networking
- [ ] Ansible
- [ ] Helm
- [ ] Prometheus
- [ ] Grafana
- [ ] Argo CD
- [ ] Security / DevSecOps
- [ ] Bash / Python scripting
- [ ] Observability
- [ ] System Design
- [ ] SRE fundamentals

---

# 📝 Documentation Standards

When adding a new topic, try to follow this structure:

```markdown
# Topic Name

## 1. What is it?

Short explanation.

## 2. Why do we use it?

Key use cases.

## 3. How does it work?

Important flow/components.

## 4. Important Commands

```bash
command
```

## 5. Practical Example

Configuration / code / hands-on example.

## 6. Troubleshooting

Common problem → investigation → solution.

## 7. Common Mistakes

Things to watch out for.

## 8. Interview Questions

### Q1. Question?

Answer.

### Q2. Question?

Answer.

## 9. Quick Revision

- Key point
- Key point
- Key point
```

---

# 🔄 Learning Workflow

```text
                 ┌──────────────┐
                 │    Learn     │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │   Practice   │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │ Troubleshoot │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │   Document   │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │    Revise    │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │   Interview  │
                 └──────┬───────┘
                        ↓
                      Repeat
```

---

# ⭐ Repository Philosophy

> **Don't just memorize commands. Understand what they do, why they are used, and how you would troubleshoot them when they fail.**

This repository is intended to become my **single-source DevOps interview preparation guide** — practical enough for hands-on learning and concise enough for quick revision.

---

## 🚀 Final Goal

```text
                    DEVOPS ENGINEER
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
       CONCEPTS         PRACTICE        TROUBLESHOOT
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                      DOCUMENT
                           ↓
                        REVISE
                           ↓
                       INTERVIEW
```

**Learn → Practice → Troubleshoot → Document → Revise → Interview → Repeat.**
