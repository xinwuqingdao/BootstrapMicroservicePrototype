# 🌐 `hello-frontend` Microservice – Angular SPA Setup & Deployment Guide

This guide explains how to build, containerize, and deploy a standalone Angular frontend that calls a Spring Boot backend (`hello-backend`) and is deployed via Kubernetes and Jenkins.

---

## 📁 Project Structure

```
hello-frontend/
├── src/                  # Angular application source
├── dist/                 # Angular build output (after build)
├── Dockerfile            # For Docker image
├── Jenkinsfile           # CI/CD pipeline for Jenkins
└── k8s/
    └── hello-frontend.yaml   # Kubernetes deployment and service manifest
```

---

## ✅ Step-by-Step Instructions

### 1️⃣ Create Angular Project (no SSR)

```bash
ng new hello-frontend --routing=false --style=css
cd hello-frontend
```

When prompted:
- ❌ Do **not** enable SSR/Prerendering
- ❌ Do **not** use standalone components (unless you want to)

---

### 2️⃣ Add Hello Service to Call Backend API

```bash
ng generate service hello
```

**src/app/hello.service.ts**
```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class HelloService {
  constructor(private http: HttpClient) {}
  getHello(): Observable<string> {
    return this.http.get('http://localhost:30080/hello', { responseType: 'text' });
  }
}
```

---

### 3️⃣ Display Backend Response in Component

**src/app/app.component.ts**
```ts
import { Component, OnInit } from '@angular/core';
import { HelloService } from './hello.service';

@Component({
  selector: 'app-root',
  template: `<h1>{{ message }}</h1>`
})
export class AppComponent implements OnInit {
  message = 'Loading...';
  constructor(private helloService: HelloService) {}
  ngOnInit() {
    this.helloService.getHello().subscribe({
      next: (msg) => this.message = msg,
      error: () => this.message = 'Error connecting to backend'
    });
  }
}
```

---

### 4️⃣ Enable HTTP in `AppModule`

**src/app/app.module.ts**
```ts
import { HttpClientModule } from '@angular/common/http';
@NgModule({
  imports: [BrowserModule, HttpClientModule],
  ...
})
```

---

### 5️⃣ Build Angular App

```bash
npm install
npm run build -- --configuration production
```

Ensure `dist/hello-frontend/index.html` exists.

---

### 6️⃣ Create `.dockerignore`

**.dockerignore**
```
node_modules
dist
.git
*.log
```

---

### 7️⃣ Dockerfile for Production Build

**Dockerfile**
```dockerfile
FROM node:18 AS build
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build -- --configuration production

FROM nginx:alpine
COPY --from=build /app/dist/hello-frontend /usr/share/nginx/html
EXPOSE 80
```

---

### 8️⃣ Run and Test Container Locally

```bash
docker build -t hello-frontend .
docker run -p 4200:80 hello-frontend
```

Access the app at: [http://localhost:4200](http://localhost:4200)

---

### 9️⃣ Kubernetes Deployment YAML

**k8s/hello-frontend.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-frontend
  template:
    metadata:
      labels:
        app: hello-frontend
    spec:
      containers:
      - name: hello-frontend
        image: hello-frontend:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hello-frontend-service
spec:
  type: NodePort
  selector:
    app: hello-frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Deploy:
```bash
kubectl apply -f k8s/hello-frontend.yaml
```

Access via: [http://localhost:30080](http://localhost:30080)

---

### 🔟 Jenkins CI/CD Pipeline

**Jenkinsfile**
```groovy
pipeline {
    agent any
    environment {
        IMAGE_NAME = "hello-frontend"
        K8S_YAML = "k8s/hello-frontend.yaml"
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-org/hello-frontend.git'
            }
        }
        stage('Install & Build') {
            steps {
                sh 'npm install'
                sh 'npm run build -- --configuration production'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f $K8S_YAML'
            }
        }
    }
}
```

---

## ✅ Notes

- Ensure CORS is enabled on the Spring Boot backend to allow requests from `http://localhost:4200` or `http://localhost:30080`
- Use `imagePullPolicy: Never` for local Docker images in Rancher Desktop or Minikube
- Use `.env` or environment.ts files if deploying to different environments (dev, staging, prod)
