# Day 14 – Networking Fundamentals & Hands-on Checks

## OSI vs TCP/IP Models

### OSI Model (L1–L7)
- A 7-layer model used to understand how data moves through a network.
- Layers: Physical, Data Link, Network, Transport, Session, Presentation, Application.

### TCP/IP Model
- Practical 4-layer networking model used on the internet.
- Layers: Link, Internet, Transport, Application.

### Protocol Placement
- IP → Internet/Network Layer
- TCP/UDP → Transport Layer
- HTTP/HTTPS → Application Layer
- DNS → Application Layer

### Real Example
`curl https://example.com`
- Application layer request using HTTP/HTTPS
- Travels over TCP
- Routed using IP

---

# Hands-on Networking Checks

## 1. Identity Check

### Command
```bash
hostname -I
Output
192.168.1.5
Observation
System IP address identified successfully.
Local machine is connected to the network.
2. Reachability Test
Command
ping google.com
Output
64 bytes from 142.250.183.14: icmp_seq=1 ttl=116 time=24 ms
64 bytes from 142.250.183.14: icmp_seq=2 ttl=116 time=26 ms
Observation
Host reachable successfully.
Low latency and 0% packet loss.
3. Path Check
Command
traceroute google.com
Observation
Multiple hops observed between local network and destination.
One hop showed timeout which can happen due to firewall restrictions.
4. Open Ports & Services
Command
ss -tulpn
Sample Output
tcp LISTEN 0 128 0.0.0.0:22
Observation
SSH service running on port 22.
System is accepting remote SSH connections.
5. DNS Resolution
Command
nslookup google.com
Output
Address: 142.250.183.14
Observation
DNS resolution working correctly.
Domain successfully mapped to IP address.
6. HTTP Header Check
Command
curl -I https://google.com
Output
HTTP/2 200
Observation
Website reachable successfully.
HTTP status code 200 means request succeeded.
7. Connections Snapshot
Command
netstat -an | head
Observation
LISTEN connections waiting for incoming requests.
ESTABLISHED connections represent active communication sessions.
Mini Task – Port Probe & Interpretation
Step 1: Identify Listening Port

Found SSH service listening on port 22.

Command
ss -tulpn
Step 2: Test Port Reachability
Command
nc -zv localhost 22
Output
Connection to localhost 22 port [tcp/ssh] succeeded!
Observation
Port 22 is reachable locally.
SSH service is active and accepting connections.
If Not Reachable

Next checks:

Verify service status
Check firewall rules
Confirm correct listening port
Reflection
Fastest Signal During Troubleshooting
ping quickly tells whether the target is reachable.
If DNS Fails
Inspect Application Layer (DNS service)
Also check Internet Layer connectivity.
If HTTP 500 Appears
Application layer issue.
Check web server logs and backend service health.
Two Follow-up Checks in Real Incident
Check firewall/security group rules.
Verify service status and logs using:
systemctl status <service>
journalctl -xe