---
layout: post
title:  "Podman Learning Plan - 4 Week Intensive"
date:   2026-01-12 10:00:00 +0800
categories: learning containers devops
published: false
---

## Overview
This 4-week plan will take you from Podman basics to production-ready containerization skills. Focus: rootless containers, security, and real-world deployments.

---

## Week 1: Podman Fundamentals

### Day 1-2: Installation & Core Concepts
- [ ] Install Podman on your machine
- [ ] Understand Docker vs Podman differences (daemonless, rootless)
- [ ] Learn about OCI (Open Container Initiative) standards
- [ ] Basic commands: `podman pull`, `podman run`, `podman ps`, `podman stop`

**Hands-on:**
```bash
podman run -it --rm alpine:latest /bin/sh
podman run -d -p 8080:80 nginx:latest
podman logs <container-id>
podman exec -it <container-id> /bin/bash
```

### Day 3-4: Building Images
- [ ] Create your first Containerfile (Dockerfile equivalent)
- [ ] Multi-stage builds
- [ ] Layer caching optimization
- [ ] `.containerignore` file

**Project:** Containerize a simple web app (Node.js/Python/Go)
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Day 5-7: Rootless Containers & Security
- [ ] Understand rootless mode benefits
- [ ] Configure user namespaces
- [ ] Security best practices (non-root users, read-only filesystems)
- [ ] Scan images for vulnerabilities: `podman scan` or Trivy

**Hands-on:**
```bash
# Run as non-root user
podman run --user 1000:1000 myapp

# Read-only root filesystem
podman run --read-only --tmpfs /tmp myapp

# Security scanning
podman scan myimage:latest
```

---

## Week 2: Networking & Storage

### Day 8-10: Container Networking
- [ ] Default bridge network
- [ ] Custom networks: `podman network create`
- [ ] Container-to-container communication
- [ ] Port mapping and exposure
- [ ] DNS resolution between containers

**Hands-on:**
```bash
podman network create mynet
podman run -d --name db --network mynet postgres:15
podman run -d --name app --network mynet -p 8080:8080 myapp
# App can reach db via hostname 'db'
```

### Day 11-14: Volumes & Persistent Storage
- [ ] Volume types: named volumes, bind mounts, tmpfs
- [ ] Volume drivers and plugins
- [ ] Data persistence strategies
- [ ] Backup and restore volumes

**Project:** Multi-container app with database persistence
```bash
# Create volume
podman volume create pgdata

# Run PostgreSQL with persistent storage
podman run -d --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Backup volume
podman run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/pgdata-backup.tar.gz /data
```

---

## Week 3: Pods & Orchestration

### Day 15-17: Podman Pods
- [ ] Understand pod concept (Kubernetes-like)
- [ ] Create and manage pods
- [ ] Shared namespaces in pods (network, IPC)
- [ ] Infra containers

**Hands-on:**
```bash
# Create a pod
podman pod create --name webapp -p 8080:80

# Add containers to pod
podman run -d --pod webapp --name nginx nginx:latest
podman run -d --pod webapp --name app myapp:latest

# Manage pod
podman pod ps
podman pod stop webapp
podman pod rm webapp
```

### Day 18-21: Podman Compose & Kubernetes
- [ ] Install `podman-compose`
- [ ] Convert Docker Compose files
- [ ] Generate Kubernetes YAML: `podman generate kube`
- [ ] Deploy to Kubernetes from Podman

**Project:** 3-tier application (web + api + db)

`docker-compose.yml`:
```yaml
version: '3'
services:
  db:
    image: postgres:15
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret
  
  api:
    build: ./api
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql://postgres:secret@db:5432/mydb
  
  web:
    build: ./web
    ports:
      - "8080:80"
    depends_on:
      - api
volumes:
  db_data:
```

```bash
podman-compose up -d
podman generate kube webapp > webapp-k8s.yaml
```

---

## Week 4: Production Readiness & Advanced Topics

### Day 22-24: CI/CD Integration
- [ ] Build images in CI pipelines
- [ ] Push to registries (Docker Hub, Quay.io, private registry)
- [ ] Automated testing in containers
- [ ] Multi-arch builds (amd64, arm64)

**GitHub Actions example:**
```yaml
name: Build and Push
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build image
        run: podman build -t myapp:${{ github.sha }} .
      
      - name: Push to registry
        run: |
          echo ${{ secrets.REGISTRY_PASSWORD }} | podman login -u ${{ secrets.REGISTRY_USER }} --password-stdin quay.io
          podman push myapp:${{ github.sha }} quay.io/myuser/myapp:${{ github.sha }}
```

### Day 25-26: Monitoring & Logging
- [ ] Container resource limits (CPU, memory)
- [ ] Health checks
- [ ] Log collection strategies
- [ ] Integration with Prometheus/Grafana (prep for observability plan)

**Hands-on:**
```bash
# Resource limits
podman run -d \
  --memory=512m \
  --cpus=0.5 \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  myapp

# View metrics
podman stats
```

### Day 27-28: Real-world Project
**Build a complete containerized application:**
- [ ] Multi-service application (at least 3 containers)
- [ ] Custom Containerfiles with optimization
- [ ] Networking between services
- [ ] Persistent storage
- [ ] Health checks and restart policies
- [ ] Environment-based configuration
- [ ] Documentation

**Example:** Microservices blog platform
- Frontend (React/Vue)
- API (Node.js/Python/Go)
- Database (PostgreSQL)
- Cache (Redis)
- Reverse proxy (Nginx)

---

## Resources

### Official Documentation
- [Podman Official Docs](https://docs.podman.io/)
- [Podman Tutorials](https://github.com/containers/podman/tree/main/docs/tutorials)

### Books & Courses
- "Podman in Action" by Daniel Walsh
- Red Hat Podman tutorials
- Linux containers deep dive

### Tools
- [Podman Desktop](https://podman-desktop.io/) - GUI for Podman
- [Trivy](https://github.com/aquasecurity/trivy) - Security scanner
- [Buildah](https://buildah.io/) - Advanced image building
- [Skopeo](https://github.com/containers/skopeo) - Image management

### Practice Projects
1. **LeetCode Tracker** (from your personal projects list)
2. **File Transfer Service** (aligns with your work)
3. **Kafka Monitor Dashboard** (aligns with your work)

---

## Weekly Goals Checklist

### Week 1
- [ ] Podman installed and configured
- [ ] Simple web app containerized
- [ ] Image built with multi-stage Containerfile
- [ ] Basic security practices applied

### Week 2
- [ ] Custom network created
- [ ] Multi-container app running
- [ ] Database with persistent volume
- [ ] Volume backup/restore performed

### Week 3
- [ ] Pod with multiple containers
- [ ] Docker Compose converted to Podman
- [ ] Kubernetes YAML generated
- [ ] 3-tier app running

### Week 4
- [ ] CI/CD pipeline created
- [ ] Images pushed to registry
- [ ] Resource limits and health checks
- [ ] Complete project deployed

---

## Next Steps
After completing this plan:
1. Move to observability (Prometheus + Grafana)
2. Implement GitOps (ArgoCD)
3. Learn Kubernetes deeper
4. Build production-grade microservices

**Start Date:** 2026-01-13
**Target Completion:** 2026-02-09

Good luck! Track your progress by checking off items daily.
