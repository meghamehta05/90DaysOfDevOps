2026/day-70/day-70-variables-loops.md

 1. Variables Demo Playbook
variables-demo.yml
---
- name: Variable demo
  hosts: all
  become: true

  vars:
    app_name: terraweek-app
    app_port: 8080
    app_dir: "/opt/{{ app_name }}"
    packages:
      - git
      - curl
      - wget

  tasks:
    - name: Print app details
      debug:
        msg: "Deploying {{ app_name }} on port {{ app_port }}"

    - name: Create directory
      file:
        path: "{{ app_dir }}"
        state: directory

    - name: Install packages
      yum:
        name: "{{ packages }}"
        state: present
 Run
ansible-playbook variables-demo.yml
 Override variable (CLI wins)
ansible-playbook variables-demo.yml -e "app_name=myapp app_port=9090"

✔ CLI variables override everything

 2. Variable Precedence (Important)

Order (low → high priority):

group_vars/all
group_vars/<group>
host_vars/<host>
playbook vars
extra vars (-e)  ← highest
 3. Folder Structure
ansible-practice/
├── inventory.ini
├── ansible.cfg
├── group_vars/
│   ├── all.yml
│   ├── web.yml
│   └── db.yml
├── host_vars/
│   └── web-server.yml
└── playbooks/
    └── site.yml
 group_vars & host_vars
group_vars/all.yml
ntp_server: pool.ntp.org
app_env: development
common_packages:
  - vim
  - htop
  - tree
group_vars/web.yml
http_port: 80
max_connections: 1000
web_packages:
  - nginx
group_vars/db.yml
db_port: 3306
db_packages:
  - mysql-server
host_vars/web-server.yml
max_connections: 2000
custom_message: "Primary web server"
 site.yml
---
- name: Common config
  hosts: all
  become: true
  tasks:
    - name: Install common packages
      yum:
        name: "{{ common_packages }}"
        state: present

    - name: Show environment
      debug:
        msg: "{{ app_env }}"

- name: Web config
  hosts: web
  tasks:
    - name: Show config
      debug:
        msg: "Port {{ http_port }}, Max {{ max_connections }}"

    - name: Host message
      debug:
        msg: "{{ custom_message }}"
 4. Ansible Facts
View facts
ansible web-server -m setup
Filter facts
ansible web-server -m setup -a "filter=ansible_os_family"
facts-demo.yml
---
- name: Facts demo
  hosts: all

  tasks:
    - name: Show system info
      debug:
        msg: >
          OS: {{ ansible_distribution }},
          RAM: {{ ansible_memtotal_mb }},
          IP: {{ ansible_default_ipv4.address }}
 5 facts used in real DevOps
ansible_distribution → OS detection
ansible_memtotal_mb → capacity planning
ansible_default_ipv4 → networking
ansible_hostname → server identity
ansible_processor_vcpus → scaling decisions
 5. Conditionals
conditional-demo.yml
---
- name: Condition demo
  hosts: all
  become: true

  tasks:
    - name: Install Nginx on web
      yum:
        name: nginx
      when: "'web' in group_names"

    - name: Install MySQL on db
      yum:
        name: mysql-server
      when: "'db' in group_names"

    - name: Low memory warning
      debug:
        msg: "Low memory!"
      when: ansible_memtotal_mb < 1024

    - name: Amazon Linux only
      debug:
        msg: "Amazon Linux detected"
      when: ansible_distribution == "Amazon"
 6. Loops
loops-demo.yml
---
- name: Loop demo
  hosts: all
  become: true

  vars:
    users:
      - name: dev1
        groups: wheel
      - name: dev2
        groups: wheel

    dirs:
      - /opt/app/logs
      - /opt/app/data

  tasks:
    - name: Create users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
      loop: "{{ users }}"

    - name: Create directories
      file:
        path: "{{ item }}"
        state: directory
      loop: "{{ dirs }}"
 loop vs with_items
loop	with_items
modern	old
recommended	deprecated
 7. Server Report Playbook
server-report.yml
---
- name: Server report
  hosts: all

  tasks:
    - name: Disk check
      command: df -h /
      register: disk

    - name: Memory check
      command: free -m
      register: mem

    - name: Report
      debug:
        msg:
          - "Host: {{ inventory_hostname }}"
          - "OS: {{ ansible_distribution }}"
          - "RAM: {{ ansible_memtotal_mb }}"
          - "Disk: {{ disk.stdout_lines[1] }}"

    - name: Save report
      copy:
        content: |
          Host: {{ inventory_hostname }}
          OS: {{ ansible_distribution }}
          RAM: {{ ansible_memtotal_mb }}
          Disk: {{ disk.stdout }}
          Time: {{ ansible_date_time.iso8601 }}
        dest: /tmp/report.txt
      become: true
 Run everything
ansible-playbook variables-demo.yml
ansible-playbook site.yml
ansible-playbook conditional-demo.yml
ansible-playbook loops-demo.yml
ansible-playbook server-report.yml
 Key Learnings
Variables make playbooks dynamic
Facts give real system intelligence
Conditionals control execution flow
Loops automate bulk operations
register captures command output