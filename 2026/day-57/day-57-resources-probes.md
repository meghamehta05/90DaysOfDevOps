1. Resource Requests vs Limits
 Requests (Scheduling)
Minimum resources guaranteed to a Pod
Used by Kubernetes scheduler to decide node placement
 Limits (Runtime Enforcement)
Maximum resources a Pod can use
Enforced by kubelet
 Example Pod
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "250m"
        memory: "256Mi"
 QoS Classes
Type	Condition
Guaranteed	requests = limits
Burstable	requests < limits
BestEffort	no requests/limits

 Your pod = Burstable

 2. OOMKilled (Memory Limit Breach)

When memory exceeds limit → container is killed instantly.

apiVersion: v1
kind: Pod
metadata:
  name: memory-stress
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
    resources:
      limits:
        memory: "100Mi"
 Result
Container crashes
Status: OOMKilled
Exit code: 137
137 = 128 + SIGKILL
 3. Pending Pod (Too many resources)

If request is too high:

resources:
  requests:
    cpu: "100"
    memory: "128Gi"

 Pod stays Pending

Scheduler event:

0/Nodes are available: insufficient cpu
 4. Liveness Probe (Restart on failure)
Detects dead/stuck container
Restarts Pod automatically
livenessProbe:
  exec:
    command: ["cat", "/tmp/healthy"]
  periodSeconds: 5
  failureThreshold: 3

 If file disappears → container restarts

 5. Readiness Probe (Traffic control)
Does NOT restart Pod
Removes Pod from Service endpoints
readinessProbe:
  httpGet:
    path: /
    port: 80
 Behavior
Event	Restart?	Traffic?
Failure	 No	          Removed
Recovery No restart	 Added back
 6. Startup Probe (Slow apps)
Gives app time to start
Disables liveness/readiness temporarily
startupProbe:
  exec:
    command: ["cat", "/tmp/started"]
  periodSeconds: 5
  failureThreshold: 12

 12 × 5s = 60s startup window

 If failureThreshold = 2
Only 10 seconds allowed
Slow apps will crash repeatedly
 Quick Comparison
Probe	Purpose	Action
Liveness	Is app alive?	Restart
Readiness	Can it serve traffic?	Remove from Service
Startup	Is app ready to start?	Delay other probes
 Key Takeaways
Requests = scheduling guarantee
Limits = hard runtime cap
CPU → throttled
Memory → killed (OOMKilled)
Probes = self-healing system