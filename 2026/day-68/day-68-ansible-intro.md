Create file:

2026/day-68/day-68-ansible-intro.md

 1. Ansible Architecture (Write this in your file)
What is Configuration Management?

Configuration management ensures servers always stay in a desired state (correct packages, configs, users, services). It removes manual setup and avoids configuration drift.

Why we need it
Automates server setup
Prevents manual errors
Ensures consistency across environments
Saves time in scaling systems
Ansible vs Chef / Puppet / Salt
Tool	Type	Requirement
Ansible	Agentless	SSH only
Chef	Agent-based	Requires node agent
Puppet	Agent-based	Requires agent
Salt	Hybrid	Agent optional

 Ansible is simplest because no software is installed on target servers

What does "Agentless" mean?

It means:

No Ansible software installed on EC2 instances
Only SSH is used for communication
Ansible Architecture
Control Node → Your laptop / EC2 where Ansible runs
Managed Nodes → Target EC2 servers
Inventory → List of servers
Modules → Tasks (install, copy, execute)
Playbooks → YAML automation scripts

 Flow:

Control Node → SSH → Managed Nodes → Executes Modules
 2. Lab Setup (EC2 Instances)

You created 3 EC2 instances:

Role	Instance
Web Server	EC2-1
App Server	EC2-2
DB Server	EC2-3

✔ Amazon Linux 2
✔ t2.micro
✔ SSH enabled (port 22)

 3. Install Ansible (Control Node)
Installed on:

 Your local machine / control node

Why only control node?

Because Ansible works in push model, not client-server model.

Installation:
sudo apt update
sudo apt install ansible -y
Verify:
ansible --version
 4. Inventory Setup
Create folder:
mkdir ansible-practice && cd ansible-practice
inventory.ini
[web]
web-server ansible_host=IP1

[app]
app-server ansible_host=IP2

[db]
db-server ansible_host=IP3

[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/your-key.pem
Test connection
ansible all -i inventory.ini -m ping

✔ Expected output:

"ping": "pong"
 5. Ad-hoc Commands
Uptime
ansible all -i inventory.ini -m command -a "uptime"
Memory check
ansible web -i inventory.ini -m command -a "free -h"
Disk usage
ansible all -i inventory.ini -m command -a "df -h"
Install package
ansible web -i inventory.ini -m yum -a "name=git state=present" --become
Copy file
echo "Hello from Ansible" > hello.txt

ansible all -i inventory.ini -m copy -a "src=hello.txt dest=/tmp/hello.txt"
Verify file
ansible all -i inventory.ini -m command -a "cat /tmp/hello.txt"
 What is --become?

 It means run as root (sudo privileges)
Used for:

Installing packages
Editing system files
Starting services
 6. Inventory Groups
[application:children]
web
app

[all_servers:children]
application
db
Test groups
ansible application -m ping
ansible db -m ping
ansible all_servers -m ping
Patterns
ansible 'web:app' -m ping
ansible 'all:!db' -m ping
 7. ansible.cfg (Important)
[defaults]
inventory = inventory.ini
host_key_checking = False
remote_user = ec2-user
private_key_file = ~/your-key.pem

Now you can run:

ansible all -m ping
 8. command vs shell
Module	Use
command	Simple commands (safe, no pipes)
shell	Supports pipes, redirects, complex commands
Example difference:
ansible all -m command -a "ls -l"
ansible all -m shell -a "ls -l | grep txt"
 9. Key Learnings
Ansible is agentless
Uses SSH only
Inventory defines infrastructure
Ad-hoc commands = quick tasks
Playbooks = automation scripts
--become = sudo access