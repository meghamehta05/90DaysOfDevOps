Step 1: Create Deployment
kubectl apply -f app-deployment.yaml
kubectl get pods -o wide
Step 2: ClusterIP Service (Internal)
kubectl apply -f clusterip-service.yaml
kubectl get services
Test it (IMPORTANT):
kubectl run test-client --image=busybox -it --rm -- sh

Inside:

wget -qO- http://web-app-clusterip
Step 3: Check DNS

Inside a pod:

nslookup web-app-clusterip

You should see ClusterIP returned.

 Step 4: NodePort Service
kubectl apply -f nodeport-service.yaml
kubectl get services
Access it:
If Minikube:
minikube service web-app-nodeport --url
If local cluster:
curl http://localhost:30080
 Step 5: LoadBalancer
kubectl apply -f loadbalancer-service.yaml
kubectl get services

If it shows:

EXTERNAL-IP: <pending>

That is NORMAL (because you're not on AWS/GCP).

Step 6: Verify routing
kubectl get endpoints
kubectl describe service web-app-clusterip