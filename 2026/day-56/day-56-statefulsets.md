What is a StatefulSet?

A StatefulSet is a Kubernetes workload used for stateful applications like databases (MySQL, MongoDB, Kafka).

Unlike Deployments, it gives:

Stable pod names (web-0, web-1, web-2)
Stable network identity
Separate storage for each pod
Ordered startup and shutdown
 Why not Deployment?

Deployments:

Pods have random names
No stable identity
No guaranteed storage per pod

Problem for DBs:

If pod restarts → identity + data can break

 StatefulSet vs Deployment
Feature	Deployment	StatefulSet
Pod names	Random	Fixed (web-0, web-1)
Startup	Parallel	Ordered
Storage	Shared/none	Per-pod PVC
Identity	None	Stable DNS
 Headless Service (IMPORTANT)

StatefulSet needs a Headless Service:

clusterIP: None

Why?

No load balancing
Gives direct DNS for each pod

Example DNS:

web-0.myservice.default.svc.cluster.local
StatefulSet Flow
Create Headless Service
Create StatefulSet

Pods are created in order:

web-0 → web-1 → web-2

Each pod gets its own PVC:

data-web-0
data-web-1
data-web-2
 Key Feature: Storage per Pod

Each pod:

Gets its own volume
Keeps data even after restart
Reattaches same PVC automatically

So:

delete pod → data still exists

 Scaling Behavior
Scale UP:
web-3 → web-4
Scale DOWN:
web-4 → web-3 (reverse order)

PVCs are NOT deleted → data safe

 Stable DNS Example
Pod	DNS
web-0	web-0.svc.cluster.local
web-1	web-1.svc.cluster.local

Each pod is individually addressable.

 Key Takeaways
StatefulSet = for databases
Deployment = for web apps
Headless Service = enables pod-level DNS
PVC = gives storage persistence per pod
Order matters in StatefulSets