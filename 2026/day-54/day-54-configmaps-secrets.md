Task 1: ConfigMap (literals)
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080

Check:

kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
Task 2: ConfigMap from file
Create file:
nano default.conf

Paste:

server {
  listen 80;

  location /health {
    return 200 "healthy";
  }
}

Create ConfigMap:

kubectl create configmap nginx-config --from-file=default.conf=default.conf

Verify:

kubectl get configmap nginx-config -o yaml
 Task 3: Use ConfigMap in Pods
Pod 1 (env vars)
apiVersion: v1
kind: Pod
metadata:
  name: busybox-env
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config

Apply:

kubectl apply -f busybox-env.yaml
kubectl logs busybox-env
Pod 2 (nginx volume mount)
apiVersion: v1
kind: Pod
metadata:
  name: nginx-config-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: nginx-config
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: nginx-config
    configMap:
      name: nginx-config

Apply:

kubectl apply -f nginx-config-pod.yaml

Test:

kubectl exec -it nginx-config-pod -- curl localhost/health
 Task 4: Secret
kubectl create secret generic db-credentials \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=s3cureP@ssw0rd

Inspect:

kubectl get secret db-credentials -o yaml

Decode password:

echo "<base64-value>" | base64 --decode
 Task 5: Secret in Pod
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: DB_USER
Secret as volume
volumes:
- name: db-secret
  secret:
    secretName: db-credentials

Mount:

volumeMounts:
- name: db-secret
  mountPath: /etc/db-credentials
  readOnly: true
 Task 6: Live update ConfigMap
kubectl create configmap live-config --from-literal=message=hello

Patch:

kubectl patch configmap live-config \
  --type merge \
  -p '{"data":{"message":"world"}}'

Watch:

kubectl exec -it <pod-name> -- cat /path/to/file
 Task 7: Cleanup
kubectl delete pod --all
kubectl delete configmap --all
kubectl delete secret --all