1. Introduction

Today I converted the AI-BankApp Kubernetes setup (12 raw YAML files) into a fully modular **Helm chart**.

The application consists of:
- Spring Boot BankApp
- MySQL database
- Ollama AI chatbot

Now the entire stack can be deployed using a single Helm command.

---

## 2. Helm Chart Overview

:contentReference[oaicite:0]{index=0} allows us to package Kubernetes manifests into reusable templates called charts.

A Helm chart for AI-BankApp includes:
- Deployments
- Services
- ConfigMaps
- Secrets
- PVCs
- HPA

---

## 3. Raw YAML vs Helm Comparison

| Feature | Raw Kubernetes YAML | Helm Chart |
|--------|--------------------|-------------|
| Files | 12 separate files | 1 structured chart |
| Configuration | Hardcoded | values.yaml driven |
| Secrets | Base64 manual | auto encoded (`b64enc`) |
| Scaling | Manual edits | HPA via values |
| Reusability | Low | High |
| Deployment | multiple kubectl apply | single helm install |

---

## 4. Helm Chart Structure


bankapp/
├── Chart.yaml
├── values.yaml
├── templates/
│ ├── configmap.yaml
│ ├── secrets.yaml
│ ├── bankapp-deployment.yaml
│ ├── mysql-deployment.yaml
│ ├── ollama-deployment.yaml
│ ├── services.yaml
│ ├── hpa.yaml
│ ├── storage.yaml
└── _helpers.tpl


---

## 5. Key values.yaml (core configuration)

```yaml
bankapp:
  replicaCount: 4
  image:
    repository: trainwithshubham/ai-bankapp-eks
    tag: latest

mysql:
  enabled: true
  persistence:
    size: 5Gi

ollama:
  enabled: true
  model: tinyllama
  persistence:
    size: 10Gi

config:
  mysqlDatabase: bankappdb

secrets:
  mysqlRootPassword: Test@123
6. Go Template Concepts Used
Concept	Usage
{{ .Values }}	Pull config from values.yaml
if	Enable/disable MySQL or Ollama
include	Reuse helper templates
toYaml	Convert structured values
nindent	Maintain YAML formatting
b64enc	Encode secrets automatically
7. Example Template Conversion
Raw YAML (ConfigMap)

Hardcoded values.

Helm Template (ConfigMap)

Uses dynamic values:

data:
  MYSQL_DATABASE: {{ .Values.config.mysqlDatabase }}
8. Helm Template Rendering

Rendered output tested using:

helm template my-bankapp bankapp/

This confirms:

All variables are replaced correctly
Conditional resources work
No syntax errors in templates
9. Feature: Toggle Components

One of the most powerful features:

ollama:
  enabled: false

 This automatically removes:

Ollama Deployment
Ollama Service
Ollama PVC
Ollama init dependencies
10. Deployment Command
helm install my-bankapp bankapp/ \
  -n bankapp --create-namespace

One command replaces:

12 kubectl apply commands
Manual secret creation
PVC setup
Service configuration
11. Key Learnings
Helm simplifies Kubernetes packaging
Templates remove duplication
values.yaml enables environment flexibility
Charts support modular architecture
Rollback and upgrades are built-in
12. Conclusion

By converting the AI-BankApp into a Helm chart, I transformed a complex multi-YAML system into a single reusable deployment package.

Now the full stack (BankApp + MySQL + Ollama) is:

Modular
Configurable
Scalable
Production-ready

