day-58-metrics-hpa.md
1. What is Metrics Server?

Metrics Server is a Kubernetes component that collects real-time CPU and memory usage from all nodes and pods.

It is required for:

kubectl top nodes
kubectl top pods
Horizontal Pod Autoscaler (HPA)

Without it → Kubernetes has no idea about actual usage.

2. Install Metrics Server
If using Minikube:
minikube addons enable metrics-server
If using Kind / other clusters:
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

If needed (common fix in local clusters):

kubectl edit deployment metrics-server -n kube-system

Add this under args:

--kubelet-insecure-tls
3. Verify Metrics Server
kubectl get pods -n kube-system | grep metrics
kubectl top nodes
kubectl top pods -A
Example Output:
CPU: 50m, 120m, etc.
Memory: 100Mi, etc.

 If this works = Metrics Server is ready

4. Deploy Sample App (HPA Target)
kubectl create deployment php-apache --image=registry.k8s.io/hpa-example

Expose it:

kubectl expose deployment php-apache --port=80

Add CPU request (IMPORTANT for HPA):

kubectl patch deployment php-apache -p \
'{"spec":{"template":{"spec":{"containers":[{"name":"hpa-example","resources":{"requests":{"cpu":"200m"}}}]}}}}'
5. Create HPA (Auto Scaling)
kubectl autoscale deployment php-apache \
--cpu-percent=50 \
--min=1 \
--max=10

Check HPA:

kubectl get hpa
kubectl describe hpa php-apache
6. Generate Load (Trigger Scaling)

Run busybox load generator:

kubectl run load-generator \
--image=busybox:1.36 \
--restart=Never \
-- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"

Watch scaling:

kubectl get hpa -w
kubectl get pods -w
7. Stop Load
kubectl delete pod load-generator

Then watch scale down slowly (takes time ~5 min)

8. What Happens Internally
Scaling Formula:
desiredReplicas =
currentReplicas × (currentCPU / targetCPU)

Example:

Target = 50%
Current = 100%
→ Pods increase
9. Key Learnings
Metrics Server = real-time CPU/memory data provider
HPA = auto scaler based on metrics
Requests are REQUIRED for HPA
Scale-up is fast, scale-down is slow (stability window)
10. kubectl commands you used
kubectl top nodes
kubectl top pods
kubectl get hpa
kubectl describe hpa
kubectl get pods -w