
---


> 📁 **File:** `README.md`  
> 📌 **Purpose:** Overview of the project

```markdown
# 🚀 Spring Boot Microservice Prototype

This project demonstrates how to containerize and deploy a Spring Boot application using Docker, Jenkins CI/CD, and Kubernetes (via Rancher Desktop).

---

## 🔧 Tech Stack

- Java 17 (Spring Boot)
- Docker
- Jenkins
- Kubernetes (Rancher Desktop)
- GitHub

---

## 🧪 Features

- REST endpoint: `/hello`
- CI/CD pipeline via Jenkins
- Local Kubernetes deployment

---

## 📚 Setup Guide

See [help.md](help.md) for full setup and deployment instructions.

---

## 📦 Build & Run (Quickstart)

```bash
./mvnw clean package
docker build -t hello-springboot .
docker run -p 8080:8080 hello-springboot

