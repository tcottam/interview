# Tekmetric Backend

A Spring Boot backend service built with Maven, containerized with Docker, and deployed via Helm on Kubernetes. The project supports both local development and CI/CD pipelines using GitHub Actions and GHCR.

---

## 📁 Project Structure

```
interview/
├── backend/
│   ├── src/                     # Java source code
│   ├── pom.xml                  # Maven configuration
│   ├── Dockerfile               # Multi-stage Docker build
│   ├── deploy/
│   │   └── chart/               # Helm chart for deployment
│   │       ├── Chart.yaml
│   │       ├── values.yaml
│   │       └── templates/
│   └── application.properties   # Spring Boot configuration
└── .github/workflows/cicd.yml   # CI/CD pipeline
```

---

## 🧱 Local Development

### Prerequisites

* Java 21 (Temurin)
* Maven 3.9+
* Docker
* Helm 3.11+
* Minikube (optional, for local K8s deploy)

### Run Locally

```bash
cd backend
mvn spring-boot:run
```

Access the app at: [http://localhost:8080](http://localhost:8080)

#### H2 Console

To access the in-memory database:

```
URL: http://localhost:8080/h2-console
JDBC URL: jdbc:h2:mem:testdb
Username: sa
Password: (leave blank or use configured value)
```

> Note: For development only. `webAllowOthers=true` is enabled via Docker for remote access within cluster.

---

## 🐳 Docker

### Build locally

```bash
cd backend
docker build -t tekmetric-backend:latest .
```

### Run locally

```bash
docker run -p 8080:8080 tekmetric-backend:latest
```

---

## ⚙️ CI/CD (GitHub Actions)

The CI/CD workflow builds, tests, lints Helm charts, and publishes a Docker image to GHCR.

```yaml
context: ./backend
file: ./backend/Dockerfile
push: true
platforms: linux/amd64,linux/arm64
tags:
  ghcr.io/<OWNER>/tekmetric-backend:<SHA>
  ghcr.io/<OWNER>/tekmetric-backend:latest
```

Secrets used:

* `GHCR_PAT`: Personal Access Token with `write:packages`

---

## ☸️ Helm Deployment (Minikube)

### Start Minikube

```bash
minikube start --driver=docker
```

### Load image from GHCR

```bash
docker login ghcr.io -u <your-username> -p <your-ghcr-pat>
```

### Deploy with Helm

```bash
helm upgrade --install tekmetric-backend backend/deploy/chart \
  --set image.repository=ghcr.io/<your-username>/tekmetric-backend \
  --set image.tag=latest
```

### Check deployment

```bash
kubectl get pods
kubectl get svc
```

Access via:

```bash
minikube service tekmetric-backend
```

---

## 🧩 Actuator & Metrics

* `/actuator/health`
* `/actuator/prometheus`

Prometheus metrics and readiness/liveness groups are explicitly configured.

---

## 🔍 Troubleshooting

**Issue:** `COPY pom.xml not found`

> Ensure Docker build context is `./backend` and the Dockerfile is referenced as `./backend/Dockerfile`.

**Issue:** `Helm lint path error`

> Always use relative path `backend/deploy/chart` from repo root in workflows.

**Issue:** Architecture mismatch on M1 Mac

> Build a multi-platform image:

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t ghcr.io/<user>/tekmetric-backend:latest .
```

---

## 🧭 Next Steps

* Add `values-ci.yaml` for CI-specific overrides.
* Automate Minikube deployment using a helper script.
* Configure ingress or service exposure for production environments.
