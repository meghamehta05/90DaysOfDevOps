1. What is Helm?

:contentReference[oaicite:0]{index=0} is a package manager for Kubernetes that simplifies application deployment by packaging Kubernetes manifests into reusable units called **charts**.

Instead of managing multiple YAML files manually, Helm lets us:
- Package Kubernetes resources into a single deployable unit
- Reuse configurations across environments (dev, staging, prod)
- Version applications and rollback easily

---

## 2. Core Concepts of Helm

### 1. Chart
A **chart** is a collection of files that define a Kubernetes application.

It includes:
- Deployments
- Services
- ConfigMaps
- Secrets
- Templates + metadata


---

### 2. Release
A **release** is a running instance of a chart in a cluster.

- Same chart can be installed multiple times
- Each install = new release name
- Each release has its own configuration

Example:
- bankapp-mysql (release 1)
- bankapp-mysql-v2 (release 2)

---

### 3. Repository
A **repository** is where charts are stored and shared.

Example:
:contentReference[oaicite:1]{index=1} Helm repo provides ready-to-use charts like MySQL, Redis, etc.

---

### 4. Values
**Values** are configuration inputs used to customize charts.

Examples:
- replica count
- image tag
- CPU/memory limits
- storage size

They allow one chart → many environments.

---

## 3. Why Helm Over Raw Kubernetes YAML?

In AI-BankApp, we had **12 YAML files**:
- Deployments
- Services
- Secrets
- PVC/PV
- ConfigMaps

### Problems with raw YAML:
- Hardcoded values
- No reuse across environments
- Manual updates required everywhere
- No versioning or rollback system



## 4. MySQL Deployment: Raw YAML vs Helm

### Raw Kubernetes approach:
- mysql-deployment.yml
- secrets.yml
- pvc.yml
- pv.yml
- service.yml

 5+ files + manual configuration

---

### Helm approach:
Single command using chart:

```bash
helm install bankapp-mysql bitnami/mysql \
  --set auth.rootPassword=Test@123 \
  --set auth.database=bankappdb

 One command handles everything:

Deployment
Service
Secret
PVC
Storage configuration
5. My mysql-values.yaml
auth:
  rootPassword: Test@123
  database: bankappdb

primary:
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi

  persistence:
    size: 5Gi
    storageClass: ""

metrics:
  enabled: true
  serviceMonitor:
    enabled: false
Explanation:
auth.rootPassword → MySQL root user password
auth.database → auto-created database
resources → CPU & memory limits for stable performance
persistence.size → storage allocated for database
storageClass → empty means default cluster storage class
metrics.enabled → enables monitoring metrics export
6. Helm Release Management
Install
helm install bankapp-mysql bitnami/mysql
Upgrade
helm upgrade bankapp-mysql bitnami/mysql --set metrics.enabled=true
Rollback
helm rollback bankapp-mysql 1
History
helm history bankapp-mysql
7. Helm Chart Structure

After pulling MySQL chart:

mysql/
 ├── Chart.yaml
 ├── values.yaml
 ├── charts/
 ├── templates/
 │    ├── deployment.yaml
 │    ├── service.yaml
 │    ├── secrets.yaml
 │    ├── statefulset.yaml
 │    └── _helpers.tpl
 └── NOTES.txt
Explanation:
Chart.yaml → metadata (name, version, app version)
values.yaml → default configuration
templates/ → Kubernetes YAML templates (with variables)
_helpers.tpl → reusable template functions
NOTES.txt → post-install instructions
8. version vs appVersion (Important)

In Chart.yaml:

version: 12.2.1
appVersion: "8.0.40"
Difference:
version → Helm chart version (changes when chart structure changes)
appVersion → actual application version (e.g., MySQL version)

 Example:

Upgrading Helm templates → change version
Upgrading MySQL engine → change appVersion
9. Helm Output Screenshots (To Add)
Helm releases:
helm list
Helm history:
helm history bankapp-mysql

(Add screenshots here)

10. Why AI-BankApp Needs Helm

The AI-BankApp has:

12 raw YAML files
Hardcoded configs
Multiple environments needed
Manual updates required
With Helm:
One reusable chart
Environment-based values files
Easier upgrades/rollbacks
Cleaner GitOps workflow
Dependency management (MySQL, Redis, etc.)