2026/day-69/day-69-playbooks.md

 1. What is a Playbook? 

A playbook is a YAML-based automation script that defines:

what to install
where to install
what state the system should be in

 It describes desired state, not steps

 Key Concepts
Term	Meaning
Play	Targets a group of hosts
Task	Single unit of work
Module	Actual action (install, copy, etc.)
Handler	Task triggered only on change
 2. First Playbook
install-nginx.yml
---
- name: Install and start Nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Create custom index page
      copy:
        content: "<h1>Deployed by Ansible - TerraWeek</h1>"
        dest: /usr/share/nginx/html/index.html
▶ Run it
ansible-playbook install-nginx.yml
 Idempotency check

Run again:

 You will see:

First run → changed
Second run → ok

 This is idempotency

 Questions Answered
1. Play vs Task
Play → defines hosts + set of tasks
Task → single action inside a play
2. Multiple plays?

 Yes, one playbook can have multiple plays

3. become: true
Play level → applies to all tasks
Task level → applies only to that task
4. If task fails?

 Remaining tasks stop for that host, but continue on others

 3. Essential Modules Playbook
essential-modules.yml
---
- hosts: all
  become: true

  tasks:

    - name: Install packages
      yum:
        name:
          - git
          - curl
          - wget
          - tree
        state: present

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Copy config file
      copy:
        src: files/app.conf
        dest: /etc/app.conf

    - name: Create directory
      file:
        path: /opt/myapp
        state: directory
        mode: '0755'

    - name: Check disk space
      command: df -h
      register: disk_output

    - name: Show disk output
      debug:
        var: disk_output.stdout_lines

    - name: Count processes
      shell: ps aux | wc -l
      register: process_count

    - name: Show process count
      debug:
        msg: "{{ process_count.stdout }}"

    - name: Set timezone
      lineinfile:
        path: /etc/environment
        line: 'TZ=Asia/Kolkata'
 Create required file
mkdir files
echo "APP_CONFIG=dev" > files/app.conf
 command vs shell
command	shell
safe	powerful
no pipes	supports pipes
recommended	use only when needed
 4. Handlers Playbook
nginx-config.yml
---
- name: Configure Nginx
  hosts: web
  become: true

  tasks:

    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Deploy config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx

    - name: Deploy index
      copy:
        content: "<h1>Server: {{ inventory_hostname }}</h1>"
        dest: /usr/share/nginx/html/index.html

    - name: Ensure running
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
 Handler behavior
Run	Handler
First run	Executes
Second run	Skips

 Only runs when changes happen

 5. Debugging & Modes
Dry run
ansible-playbook install-nginx.yml --check
Diff mode
ansible-playbook nginx-config.yml --check --diff
Verbose
ansible-playbook install-nginx.yml -vvv
Limit hosts
ansible-playbook install-nginx.yml --limit web
 6. Multi-play Playbook
multi-play.yml
---
- name: Web servers
  hosts: web
  become: true
  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

- name: App servers
  hosts: app
  become: true
  tasks:
    - name: Install build tools
      yum:
        name:
          - gcc
          - make
        state: present

- name: DB servers
  hosts: db
  become: true
  tasks:
    - name: Install MySQL
      yum:
        name: mysql
        state: present
 Run
ansible-playbook multi-play.yml
Key Learnings
Playbook = automation blueprint
Tasks run sequentially
Modules perform actions
Handlers run only when triggered
Idempotency = safe re-runs
