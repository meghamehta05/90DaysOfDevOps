nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
📄 busybox-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
📄 redis-pod.yaml (3rd Pod – custom)
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
  labels:
    app: redis
    environment: test
    team: devops
spec:
  containers:
  - name: redis
    image: redis:latest
    ports:
    - containerPort: 6379
📄 day-51-pods.md
# Day 51 – Kubernetes Pods

---

## 1. Kubernetes Manifest Structure

Every Kubernetes YAML has 4 main parts:

- apiVersion → API version used (v1 for pods)
- kind → Type of resource (Pod, Deployment, Service)
- metadata → Name, labels, identity
- spec → Desired state (containers, images, ports)

---

## 2. My Pods Created

### Nginx Pod
- Runs nginx web server
- Exposes port 80

### BusyBox Pod
- Runs a shell command
- Keeps running using sleep

### Redis Pod
- Runs Redis database
- Has multiple labels (app, environment, team)

---

## 3. Imperative vs Declarative

### Imperative
- kubectl run creates resources directly
- Faster but not reusable

### Declarative
- kubectl apply -f YAML
- Best practice for production

---

## 4. Key Difference

Imperative → "Do this now"
Declarative → "This is what I want"

---

## 5. What happens when Pod is deleted?

- Standalone Pods are not recreated
- They are permanently removed
- No controller exists to maintain them

---

## 6. Key Learnings

- Pods are the smallest deployable unit
- Each Pod can run one or more containers
- Labels help in filtering and organizing resources
- YAML structure must be exact (indentation matters)
 PUSH COMMAND

Run this in your repo:

git add .
git commit -m "Day 51 Kubernetes Pods completed"
git push origin main
 What  Just Learned


Write Kubernetes YAML from scratch
Deploy Pods manually
Debug containers using logs & exec
Use labels for filtering
Understand imperative vs declarative workflows

This is core Kubernetes foundation level skill.