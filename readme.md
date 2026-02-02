# DevOps Technical Test

## Overview

This repository demonstrates a simple but complete DevOps setup covering:

* **VM-based application deployment using Ansible + PM2**
* **Containerized application deployment using Kubernetes (Minikube)**
* **CI/CD pipeline using GitHub Actions**
* **Docker image registry using DigitalOcean Container Registry**

The goal is to show clear separation of concerns between **infrastructure provisioning**, **application runtime**, and **continuous delivery**.

---

## Architecture

There are **two application runtimes**:

### 1. VM-based Node.js App (Ansible + PM2)

* Runs directly on a Linux VM (DigitalOcean Droplet)
* Provisioned using **Ansible**
* Managed at runtime using **PM2**
* Intended to represent a traditional server-based deployment

### 2. Kubernetes-based Node.js App

* Containerized using **Docker**
* Deployed to **Minikube** running on a droplet
* Image stored in **DigitalOcean Container Registry**
* Deployed via **CI/CD pipeline**

---

## Repository Structure

```
.
├── ansible/              # Ansible playbooks and inventory
│   ├── inventory.ini
│   └── playbook.yml
├── app/                  # Node.js application (used by both runtimes)
│   ├── index.js
│   └── package.json
├── k8s/                  # Kubernetes manifests
│   ├── deployment.yaml
│   └── service.yaml
├── .github/workflows/    # GitHub Actions CI/CD pipeline
│   └── cicd.yml
├── Dockerfile
└── README.md
```

---

## Ansible (VM-based Deployment)

### Purpose

Ansible is used for **one-time provisioning and configuration** of the VM:

* Install system dependencies (git, Node.js)
* Install and configure PM2
* Clone application repository
* Install Node.js dependencies
* Run the application via PM2 with auto-start enabled

### Why Ansible is not in CI/CD

Ansible is intentionally **not part of the CI/CD pipeline** because:

* It performs **infrastructure and host configuration**, not application delivery
* Provisioning is typically a **one-time or infrequent operation**
* CI/CD focuses on **application lifecycle**, not server setup

### How to Run

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
```

The playbook is **idempotent**, meaning it can be safely re-run without breaking the system.

---

## PM2 Runtime

* Application is run under the `ubuntu` user
* PM2 is configured with `systemd` startup
* Processes are user-scoped (not global)

Check status:

```bash
pm2 status
```

---

## Kubernetes Deployment

### Components

* **Deployment**: Runs the Node.js container
* **Service**: Exposes the application

### Local Access (Minikube)

```bash
kubectl port-forward service/node-app 8080:8080
```

---

## CI/CD Pipeline

### Trigger

* Runs automatically on push to `main` branch

### Pipeline Steps

1. Checkout source code
2. Authenticate to DigitalOcean Container Registry
3. Build Docker image
4. Push image to registry
5. SSH into droplet
6. Update Kubernetes deployment using `kubectl set image`

### Why SSH is Used

* Kubernetes cluster (Minikube) runs inside a private VM
* GitHub Actions cannot directly access the cluster API
* SSH provides a secure and simple deployment mechanism

---

## Design Decisions

* **Clear separation of concerns**:

  * Ansible → infrastructure & VM runtime
  * CI/CD → container lifecycle
* **Idempotent automation** to ensure reliability
* **Minimal tooling** to keep the solution easy to reason about

---

## Assumptions & Notes

* Repository is public
* DigitalOcean credentials are stored as GitHub Secrets
* This setup is designed for demonstration/testing purposes

---

## Conclusion

This project demonstrates:

* Practical DevOps workflow
* Infrastructure automation with Ansible
* Containerization and Kubernetes deployment
* Secure and automated CI/CD pipeline

The setup intentionally mirrors real-world DevOps trade-offs rather than over-engineering the solution.