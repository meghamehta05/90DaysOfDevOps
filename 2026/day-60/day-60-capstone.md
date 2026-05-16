1. Namespace Setup (Core Isolation)
kubectl create namespace capstone
kubectl config set-context --current --namespace=capstone
 2. MySQL Deployment (Stateful + Storage + Secret)
Secret
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: capstone
stringData:
  MYSQL_ROOT_PASSWORD: rootpass
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: wppass123
kubectl apply -f mysql-secret.yaml
Headless Service (Stable DNS)
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: capstone
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: capstone
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        envFrom:
        - secretRef:
            name: mysql-secret
        resources:
          requests:
            cpu: "250m"
            memory: "512Mi"
          limits:
            cpu: "500m"
            memory: "1Gi"
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
Check MySQL
kubectl get pods
kubectl exec -it mysql-0 -- mysql -u root -prootpass -e "SHOW DATABASES;"
 3. WordPress Deployment
ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: wp-config
  namespace: capstone
data:
  WORDPRESS_DB_HOST: mysql-0.mysql.capstone.svc.cluster.local:3306
  WORDPRESS_DB_NAME: wordpress
Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: capstone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: wp-config
        env:
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_USER
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        readinessProbe:
          httpGet:
            path: /wp-login.php
            port: 80
          initialDelaySeconds: 20
        livenessProbe:
          httpGet:
            path: /wp-login.php
            port: 80
          initialDelaySeconds: 30
 4. Expose WordPress (NodePort)
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: capstone
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
 Run Everything
kubectl apply -f .
kubectl get all -n capstone
 Access WordPress
Minikube:
minikube service wordpress -n capstone
Or:
http://localhost:30080
 5. Self-Healing Test
kubectl delete pod mysql-0 -n capstone
kubectl delete pod <wordpress-pod> -n capstone
kubectl get pods -w

✔ Pods automatically recreate
✔ WordPress still works

 6. HPA (Auto Scaling)
kubectl autoscale deployment wordpress \
--cpu-percent=50 --min=2 --max=10 -n capstone