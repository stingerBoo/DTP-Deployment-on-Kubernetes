# Parasoft DTP Deployment on Kubernetes

> **Disclaimer:** Example deployment configuration for reference purposes only.

This repository provides Kubernetes manifests to deploy a **Parasoft DTP (Development Testing Platform)** environment (namespace: `dtp-2026-2`), including the DTP Server, Data Collector, Enterprise Pack, MySQL 8, and MongoDB 8. 

## Key Prerequisites

* Running Kubernetes cluster with `kubectl` configured.
* JDBC Driver (`mysql-connector-java-8.0.28.jar`) in `dockerDTP/`.
* Internal SSL certificates signed by your IT department placed in `dockerDTP/_cert2026/`.

## Repository Structure

```text
.
├── 01-dtp-base.yaml                 # Namespace, RBAC permissions, PersistentVolumeClaim, and ConfigMap (PSTRootConfig.xml)
├── 02-dtp-dbs.yaml                  # Deployments and Services for MySQL 8 and MongoDB 8
├── 03-dtp-app.yaml                  # StatefulSet and Services for DTP, Data Collector, and Extension Designer
├── dockerDTP/                       # Build context for the custom DTP image
│   ├── Dockerfile
│   ├── mysql-connector-java-8.0.28.jar
│   ├── server.xml
│   ├── ssl.config.js
│   └── _cert2026/
│       ├── <your-cert>.crt
│       └── <your-key>.key
└── dockerMysql/                     # Build context for the custom MySQL image
    ├── Dockerfile
    └── create.sql
```

## Fresh Build & Deployment Instructions

After cloning or downloading this repository, run the following commands from the root project directory.

### 1. Build Local Docker Images

```powershell
# Navigate to the root project directory containing YAML files and the dockerDTP and dockerMysql subfolders
cd <path-to-repository>

# Build a custom DTP server image (including the MySQL driver, SSL certificates, and configuration files)
docker build -t k8s/dtp:2026.2.0 ./dockerDTP

# Build a custom MySQL 8 database image (including the DTP database initialization script)
docker build -t k8s/mysql:8.0 ./dockerMysql

# Verify that both local images have been properly saved in Docker Desktop
docker images k8s/*
```


### 2. Deploy Services to Kubernetes

```powershell
# Create base resources: Namespace, RBAC permissions, persistent PVC disk, and ConfigMap with PSTRootConfig.xml
kubectl apply -f .\01-dtp-base.yaml

# Start MySQL and MongoDB databases
kubectl apply -f .\02-dtp-dbs.yaml

# Start the DTP application, Extension Designer, and Data Collector (along with services and ports 8080, 8443, 8082, 8314, 2002)
kubectl apply -f .\03-dtp-app.yaml

# Monitor the startup status of all pods in real time (target: dtp-0 2/2 Running, mysql 1/1 Running, mongodb 1/1 Running)
kubectl get pods -n dtp-2026-2 -w
```

Once `dtp-0` reaches `2/2 Running` (typically 60–90 seconds after initial database schema creation), access the web interface at:

```text
https://<your-configured-hostname>:8443
```

## Maintenance, Diagnostics & Upgrades

### Diagnostics

```powershell
# Check all running pods, services, PVC disks, and StatefulSets in the dtp-2026-2 namespace
kubectl get all,pvc -n dtp-2026-2

# View logs of the main DTP server (useful for checking Tomcat startup or database connection)
kubectl logs dtp-0 -n dtp-2026-2 -c dtp --tail=50

# View logs of Extension Designer and Data Collector
kubectl logs dtp-0 -n dtp-2026-2 -c extension-designer --tail=50

# Verify in DTP logs whether the license properly detected Kubernetes and generated the machineId
kubectl exec -it dtp-0 -n dtp-2026-2 -c dtp -- grep -iE "machineId|kubernetes|PARASOFT_POD" /usr/local/parasoft/data/logs/tests.log
```

### Clean Reset (Preserving License `machineId`)

```powershell
# Delete all pods, databases, services, PVC disk, ConfigMap, and RBAC permissions, WHILE PRESERVING the Namespace itself (unchanged UUID = same license)
kubectl delete all,pvc,cm,sa,role,rolebinding --all -n dtp-2026-2

# Verify that the PVC disk and all resources are gone before a clean redeploy
kubectl get all,pvc,cm -n dtp-2026-2
```

### Upgrading to a Newer DTP Version

```powershell
# 1. After changing the base version in dockerDTP/Dockerfile, build a new image with the new version tag
docker build -t k8s/dtp:2026.3.0 ./dockerDTP

# 2. After updating the image version number in 03-dtp-app.yaml, deploy the new version (without losing PVC data or changing machineId)
kubectl apply -f .\03-dtp-app.yaml

# 3. Monitor the restart of the dtp-0 pod on the new image version
kubectl get pods -n dtp-2026-2 -w
```
