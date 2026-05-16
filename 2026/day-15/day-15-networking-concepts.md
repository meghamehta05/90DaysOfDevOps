# Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

---

# Task 1 – DNS: How Names Become IPs

## 1. What happens when we type google.com in a browser?

- The browser sends a DNS request to find the IP address of `google.com`.
- DNS servers resolve the domain name into an IP address.
- The browser connects to that IP using TCP/IP.
- HTTP/HTTPS request is then sent to load the website.

---

## 2. DNS Record Types

| Record Type | Purpose |
|-------------|---------|
| A | Maps a domain name to an IPv4 address |
| AAAA | Maps a domain name to an IPv6 address |
| CNAME | Creates an alias from one domain to another |
| MX | Specifies mail servers for email handling |
| NS | Identifies authoritative DNS servers |

---

## 3. dig google.com

### Command
```bash
dig google.com
Sample Output
google.com.    300    IN    A    142.250.183.14
Observation
A Record IP: 142.250.183.14
TTL: 300
Task 2 – IP Addressing
1. What is IPv4?
IPv4 is a 32-bit address used to identify devices on a network.
It is divided into 4 octets separated by dots.
Example: 192.168.1.10
2. Public vs Private IP
Type	Description	Example
Public IP	Accessible over the internet	8.8.8.8
Private IP	Used inside local/private networks	192.168.1.5
3. Private IP Ranges
10.0.0.0 – 10.255.255.255
172.16.0.0 – 172.31.255.255
192.168.0.0 – 192.168.255.255
4. ip addr show
Command
ip addr show
Observation
Private IP identified: 192.168.1.5
Belongs to private range 192.168.x.x
Task 3 – CIDR & Subnetting
1. Meaning of /24
/24 means first 24 bits are used for network portion.
Remaining 8 bits are used for hosts.
2. Usable Hosts
CIDR	Usable Hosts
/24	254
/16	65,534
/28	14
3. Why Do We Subnet?
Subnetting helps divide large networks into smaller manageable networks.
Improves security, performance, and efficient IP usage.
4. CIDR Table
CIDR	Subnet Mask	Total IPs	Usable Hosts
/24	255.255.255.0	256	254
/16	255.255.0.0	65,536	65,534
/28	255.255.255.240	16	14
Task 4 – Ports: The Doors to Services
1. What is a Port?
Ports identify specific services running on a machine.
They allow multiple applications to communicate over the same IP address.
2. Common Ports
Port	Service
22	SSH
80	HTTP
443	HTTPS
53	DNS
3306	MySQL
6379	Redis
27017	MongoDB
3. ss -tulpn
Command
ss -tulpn
Sample Observation
Port	Service
22	SSH
3306	MySQL
Task 5 – Putting It Together
1. curl http://myapp.com:8080
DNS resolves myapp.com to an IP address.
TCP connection is created to port 8080.
HTTP request is sent over IP networking.
2. App Can't Reach Database at 10.0.1.50:3306

First checks:

Verify database service is running.
Check network connectivity using ping or telnet.
Verify firewall/security group rules for port 3306.