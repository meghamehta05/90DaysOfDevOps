1. What is Helm?

Helm is the package manager for Kubernetes.

Instead of writing multiple YAML files (Deployment, Service, ConfigMap, etc.), Helm packages everything into a Chart.

Key Idea:

Helm = apt/npm for Kubernetes

2. Core Concepts
Chart → A Kubernetes application package (templates + config)
Release → A deployed instance of a chart
Repository → A collection of charts (like Docker Hub for Kubernetes apps)
3. Install Helm
Check installation:
helm version
Check environment:
helm env

 This confirms Helm is installed correctly.

4. Add Repository (Bitnami)
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

Search charts:

helm search repo nginx
helm search repo bitnami

 Bitnami contains hundreds of production-ready charts.

5. Install a Chart
helm install my-nginx bitnami/nginx

Check Kubernetes objects:

kubectl get all

Check Helm release:

helm list
helm status my-nginx
6. Customize Helm with Values
Install with overrides:
helm install my-nginx bitnami/nginx \
--set replicaCount=3 \
--set service.type=NodePort
Create custom-values.yaml
replicaCount: 3

service:
  type: NodePort

resources:
  limits:
    cpu: 200m
    memory: 256Mi

Install using file:

helm install my-custom-nginx bitnami/nginx -f custom-values.yaml

Check values:

helm get values my-custom-nginx
7. Upgrade and Rollback
Upgrade release:
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
Check history:
helm history my-nginx
Rollback:
helm rollback my-nginx 1

 Helm keeps revision history like Git.

8. Create Your Own Helm Chart
Scaffold chart:
helm create my-app
Structure:
my-app/
 ├── Chart.yaml
 ├── values.yaml
 ├── templates/
Key templating examples:
replicas: {{ .Values.replicaCount }}
image: {{ .Values.image.repository }}
name: {{ .Chart.Name }}
release: {{ .Release.Name }}
Validate chart:
helm lint my-app
Render YAML (dry run):
helm template my-release ./my-app
Install chart:
helm install my-release ./my-app
Upgrade chart:
helm upgrade my-release ./my-app --set replicaCount=5
9. Cleanup
helm uninstall my-nginx
helm uninstall my-custom-nginx
helm uninstall my-release

Check:

helm list
10. Key Learnings
Helm reduces hundreds of YAML files into one package
Charts are reusable templates
Releases track deployment versions
Values files allow customization without changing templates
Rollbacks are built-in and fast
11. Final Summary

Helm simplifies Kubernetes deployments by packaging manifests into reusable charts. It enables easy installation, upgrades, customization, and rollback of applications using a single command.