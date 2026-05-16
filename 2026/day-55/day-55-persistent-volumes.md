Task 1: emptyDir (prove data loss)
Pod YAML
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo $(date) > /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    emptyDir: {}
Run
kubectl apply -f emptydir-pod.yaml
kubectl exec -it emptydir-pod -- cat /data/message.txt
kubectl delete pod emptydir-pod

Recreate → file is gone ✔

 Task 2: PersistentVolume (PV)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/k8s-pv-data
kubectl apply -f pv.yaml
kubectl get pv

Status should be: Available

 Task 3: PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
kubectl apply -f pvc.yaml
kubectl get pvc
kubectl get pv

 Both should be Bound

 Task 4: Use PVC in Pod
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo $(date) >> /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
kubectl apply -f pvc-pod.yaml
kubectl exec -it pvc-pod -- cat /data/message.txt

Delete and recreate → data still there ✔

 Task 5: StorageClass
kubectl get storageclass

 Look for default like:

standard
gp2 (AWS)
default
 Task 6: Dynamic Provisioning
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
kubectl apply -f dynamic-pvc.yaml
kubectl get pv

 New PV auto-created

 Task 7: Cleanup
kubectl delete pod pvc-pod
kubectl delete pvc my-pvc dynamic-pvc
kubectl delete pv my-pv