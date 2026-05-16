1. What is an Ansible Role?

A role = structured automation package

Instead of one big playbook, everything is split into:

tasks → what to do
handlers → restart services
templates → dynamic configs
vars/defaults → variables
files → static files
meta → dependencies

 Roles = reusable + production-grade Ansible

 2. Role Structure
roles/webserver/
├── tasks/
├── handlers/
├── templates/
├── defaults/
├── vars/
├── files/
└── meta/
Generate role
ansible-galaxy init roles/webserver
 Difference: defaults vs vars
Type	Priority	Use
defaults	lowest	overridable settings
vars	high	fixed internal values

 Rule:

defaults → flexible
vars → strict
 3. Jinja2 Templates
templates/nginx-vhost.conf.j2
server {
    listen {{ http_port | default(80) }};
    server_name {{ ansible_hostname }};

    root /var/www/{{ app_name }};

    access_log /var/log/nginx/{{ app_name }}.log;
    error_log /var/log/nginx/{{ app_name }}_error.log;
}
 Template Playbook
template-demo.yml
---
- name: Deploy Nginx with template
  hosts: web
  become: true

  vars:
    app_name: terraweek-app
    http_port: 80

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Create web root
      file:
        path: "/var/www/{{ app_name }}"
        state: directory

    - name: Deploy template
      template:
        src: templates/nginx-vhost.conf.j2
        dest: "/etc/nginx/conf.d/{{ app_name }}.conf"
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
 Run with diff
ansible-playbook template-demo.yml --diff
 4. Custom Role (Webserver)
defaults/main.yml
http_port: 80
app_name: myapp
max_connections: 512
tasks/main.yml
---
- name: Install Nginx
  yum:
    name: nginx
    state: present

- name: Deploy vhost
  template:
    src: vhost.conf.j2
    dest: /etc/nginx/conf.d/{{ app_name }}.conf
  notify: Restart Nginx

- name: Create web root
  file:
    path: "/var/www/{{ app_name }}"
    state: directory

- name: Deploy index page
  template:
    src: index.html.j2
    dest: "/var/www/{{ app_name }}/index.html"

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: true
handlers/main.yml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
templates/index.html.j2
<h1>{{ app_name }}</h1>
<p>Host: {{ ansible_hostname }}</p>
<p>IP: {{ ansible_default_ipv4.address }}</p>
<p>Environment: {{ app_env | default('dev') }}</p>
 Role Playbook
site.yml
---
- name: Web servers
  hosts: web
  become: true
  roles:
    - role: webserver
      vars:
        app_name: terraweek
        http_port: 80
 Run role
ansible-playbook site.yml
 5. Ansible Galaxy
Install role
ansible-galaxy install geerlingguy.docker
Verify
ansible-galaxy list
Use role
---
- name: Install Docker
  hosts: app
  become: true
  roles:
    - geerlingguy.docker
requirements.yml
roles:
  - name: geerlingguy.docker
    version: "7.4.1"
  - name: geerlingguy.ntp
Install all
ansible-galaxy install -r requirements.yml
 6. Ansible Vault
Create encrypted file
ansible-vault create group_vars/db/vault.yml

Add:

vault_db_password: Secret123
vault_root_password: Root123
View encrypted file
ansible-vault view group_vars/db/vault.yml
Run playbook with vault
ansible-playbook db.yml --ask-vault-pass
Better (CI/CD way)
echo "mypassword" > .vault_pass
chmod 600 .vault_pass
# ansible.cfg
vault_password_file = .vault_pass
 Why Vault matters
Protects passwords
Encrypts API keys
Prevents leaks in GitHub
Required in production DevOps
 7. Combined Production Playbook
---
- name: Web servers
  hosts: web
  roles:
    - webserver

- name: App servers
  hosts: app
  roles:
    - geerlingguy.docker

- name: DB servers
  hosts: db
  tasks:
    - name: DB config
      template:
        src: templates/db-config.j2
        dest: /etc/db.conf
 DB Template
DB_HOST={{ ansible_default_ipv4.address }}
DB_PORT={{ db_port | default(3306) }}
DB_PASSWORD={{ vault_db_password }}
 Key Learnings
Roles = production structure
Templates = dynamic configs
Galaxy = reusable community automation
Vault = secret protection layer