# Tekmetric Backend - Deployment Guide

This guide explains how to build, containerize, and deploy the Tekmetric backend Spring Boot application to Kubernetes using the provided Helm chart.

## Contents
- Dockerfile
- Helm chart: `deploy/chart/tekmetric-backend`
- Example `data.sql` seed file location: `src/main/resources/data.sql`

## Prerequisites
- Java 21 and Maven (for local build)
- Docker
- kubectl
- Helm (v3+)
- kind or minikube (for local cluster testing)

## Local build & run (dev)
```bash
# build
cd backend
mvn package

# run
java -jar target/interview-1.0-SNAPSHOT.jar

# verify
curl http://localhost:8080/api/welcome
```

## Build Docker image
```bash
# from repo root where Dockerfile is located (backend/)
docker build -t tekmetric-backend:local .

docker run --rm -p 8080:8080 tekmetric-backend:local
# verify
curl http://localhost:8080/api/welcome
```

## Deploy to a local cluster (kind)
```bash
# create cluster
kind create cluster --name tekmetric

# load local image into kind
kind load docker-image tekmetric-backend:local --name tekmetric

# install helm chart
helm install tekmetric-backend deploy/chart/tekmetric-backend

# check pods
kubectl get pods

# port-forward service for testing
kubectl port-forward svc/tekmetric-backend 8080:8080
curl http://localhost:8080/api/customers
```

## Deploy to minikube
```bash
minikube start
# build image inside minikube docker daemon
eval $(minikube docker-env)
docker build -t tekmetric-backend:local .

helm install tekmetric-backend deploy/chart/tekmetric-backend
minikube service tekmetric-backend --url
```

## Observability (Prometheus)
1. Install the kube-prometheus-stack chart (Prometheus & Grafana):
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
```
2. Create a ServiceMonitor to scrape the Spring Boot `/actuator/prometheus` endpoint (if using Prometheus Operator). See README in repo for example ServiceMonitor YAML.

## CRUD API endpoints (examples)
```bash
# list
curl http://localhost:8080/api/customers

# get
curl http://localhost:8080/api/customers/1

# create
curl -X POST -H "Content-Type: application/json" -d '{"name":"Eve","email":"eve@example.com"}' http://localhost:8080/api/customers

# update
curl -X PUT -H "Content-Type: application/json" -d '{"name":"Eve Updated","email":"eve2@example.com"}' http://localhost:8080/api/customers/3

# delete
curl -X DELETE http://localhost:8080/api/customers/3
```

## Notes & Best Practices
- The Helm chart values use `tekmetric-backend:latest` by default. For production, push to a registry and pin an immutable tag (or SHA).
- Ensure `management.endpoints.web.exposure.include` includes `prometheus` and `health` endpoints in `application.properties`.
- Use resource requests/limits and non-root image.
- Protect actuator endpoints in production (network policies, auth)

## PR Checklist
- Dockerfile included and tested
- Helm chart included under `deploy/chart/tekmetric-backend`
- `src/main/resources/data.sql` added with seed data
- README updated with build & deploy steps

---