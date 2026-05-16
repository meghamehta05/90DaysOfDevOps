day-50-k8s-setup.md
# Day 50 – Kubernetes Setup & Architecture

---

## 1. Kubernetes Story (In My Words)

Kubernetes was created to solve the problem of managing large numbers of containers across multiple machines. Docker can run containers, but it cannot efficiently scale, restart failed containers, or manage workloads across a cluster. Kubernetes automates all of this.

It was originally created by Google based on their internal system called Borg, and is now maintained by the Cloud Native Computing Foundation (CNCF). The word "Kubernetes" means "helmsman" or "pilot," symbolizing that it steers containerized applications.

---

## 2. Kubernetes Architecture

### Control Plane (Master Node)

- API Server → Entry point for all commands (kubectl talks here)
- etcd → Stores all cluster state (key-value database)
- Scheduler → Assigns pods to worker nodes
- Controller Manager → Ensures desired state matches actual state

---

### Worker Node

- kubelet → Agent that manages pods on the node
- kube-proxy → Handles networking and communication
- Container Runtime → Runs containers (Docker/containerd)

---

## Flow of kubectl apply

1. User runs `kubectl apply -f pod.yaml`
2. Request goes to API Server
3. API Server stores state in etcd
4. Scheduler assigns pod to a node
5. kubelet on node creates the container
6. kube-proxy manages networking

---

## If Something Fails

- If API server goes down → cluster becomes unmanaged (no new changes)
- If worker node goes down → pods are rescheduled on other nodes

---

## 3. Tool Used

I used **kind (Kubernetes in Docker)** because:
- It runs Kubernetes inside Docker containers
- Lightweight and fast
- Perfect for local development and learning

---

## 4. Cluster Commands Used

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
kubectl get pods -n kube-system
5. kube-system Pods Explanation
kube-apiserver → Handles API requests
etcd → Stores cluster state
kube-scheduler → Assigns workloads to nodes
kube-controller-manager → Maintains cluster state
coredns → Provides DNS inside cluster
kube-proxy → Handles networking rules
6. kubeconfig

A kubeconfig is a configuration file that contains cluster details, credentials, and context information needed for kubectl to connect to a Kubernetes cluster.

It is stored at:

~/.kube/config

It allows switching between multiple clusters easily.

7. Key Learning
Kubernetes automates container orchestration
Control plane manages decisions, worker nodes execute them
kubectl is the bridge between user and cluster
Everything in Kubernetes is declarative (desired state)

---

#  FINAL PUSH COMMAND

Run this inside your repo:

```bash id="push50"
git add .
git commit -m "Day 50 Kubernetes setup completed"
git push origin main