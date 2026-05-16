1. Project Structure
ansible-docker-project/
├── ansible.cfg
├── inventory.ini
├── site.yml
├── .vault_pass
├── group_vars/
│   ├── all.yml
│   └── web/
│       └── vault.yml   (encrypted)
└── roles/
    ├── common/
    ├── docker/
    └── nginx/
 2. Inventory
[web]
web1 ansible_host=<PUBLIC_IP>
 3. ansible.cfg
[defaults]
inventory = inventory.ini
host_key_checking = False
vault_password_file = .vault_pass
4. Global Variables
group_vars/all.yml
timezone: Asia/Kolkata
project_name: devops-app
app_env: development

common_packages:
  - vim
  - curl
  - git
  - wget
  - htop
5. Vault (Docker Credentials)
ansible-vault create group_vars/web/vault.yml
vault_docker_username: your_user
vault_docker_password: your_token
 6. COMMON ROLE
roles/common/tasks/main.yml
- name: Install base packages
  yum:
    name: "{{ common_packages }}"
    state: present

- name: Set timezone
  timezone:
    name: "{{ timezone }}"

- name: Create deploy user
  user:
    name: deploy
    groups: wheel
 7. DOCKER ROLE
defaults/main.yml
docker_app_image: nginx
docker_app_tag: latest
docker_app_name: myapp
docker_app_port: 8080
container_port: 80
tasks/main.yml
- name: Install Docker
  yum:
    name:
      - docker
    state: present

- name: Start Docker
  service:
    name: docker
    state: started
    enabled: true

- name: Add deploy user to docker group
  user:
    name: deploy
    groups: docker
    append: yes

- name: Docker login
  community.docker.docker_login:
    username: "{{ vault_docker_username }}"
    password: "{{ vault_docker_password }}"
  become_user: deploy
  when: vault_docker_username is defined

- name: Pull image
  community.docker.docker_image:
    name: "{{ docker_app_image }}"
    tag: "{{ docker_app_tag }}"
    source: pull

- name: Run container
  community.docker.docker_container:
    name: "{{ docker_app_name }}"
    image: "{{ docker_app_image }}:{{ docker_app_tag }}"
    state: started
    restart_policy: always
    ports:
      - "{{ docker_app_port }}:{{ container_port }}"

- name: Health check
  uri:
    url: "http://localhost:{{ docker_app_port }}"
    status_code: 200
  register: result
  retries: 5
  delay: 3
  until: result.status == 200
 8. NGINX ROLE
defaults/main.yml
nginx_http_port: 80
nginx_upstream_port: 8080
nginx_server_name: _
tasks/main.yml
- name: Install Nginx
  yum:
    name: nginx
    state: present

- name: Remove default config
  file:
    path: /etc/nginx/conf.d/default.conf
    state: absent

- name: Deploy reverse proxy
  template:
    src: app-proxy.conf.j2
    dest: /etc/nginx/conf.d/app.conf
  notify: Reload Nginx

- name: Test config
  command: nginx -t
  changed_when: false

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: true
handlers/main.yml
- name: Reload Nginx
  service:
    name: nginx
    state: reloaded
templates/app-proxy.conf.j2
upstream app {
    server 127.0.0.1:{{ nginx_upstream_port }};
}

server {
    listen {{ nginx_http_port }};
    server_name {{ nginx_server_name }};

    location / {
        proxy_pass http://app;
    }

{% if app_env == "production" %}
    access_log /var/log/nginx/prod.log;
{% else %}
    access_log /var/log/nginx/dev.log;
{% endif %}
}
9. MASTER PLAYBOOK
site.yml
- name: Common setup
  hosts: web
  become: true
  roles:
    - common

- name: Docker setup
  hosts: web
  become: true
  roles:
    - docker

- name: Nginx setup
  hosts: web
  become: true
  roles:
    - nginx
10. RUN PROJECT
Step 1: Dry run
ansible-playbook site.yml --check --diff
Step 2: Deploy
ansible-playbook site.yml
 11. VERIFY OUTPUT
Check container
docker ps
Check app (port 8080)
curl http://localhost:8080
Check nginx (port 80)
curl http://localhost
12. TEST IDEMPOTENCY

Run again:

ansible-playbook site.yml

✔ Result should be:

mostly ok
no unnecessary changes
 13. VAULT BENEFITS
Docker credentials hidden
Not visible in Git
Safe for CI/CD pipelines
 14. ARCHITECTURE
Ansible Controller
        ↓
  Web Server (EC2)
        ↓
 Nginx (Port 80)
        ↓
 Docker Container (Port 8080)
 15. BONUS: CHANGE APP
ansible-playbook site.yml -e \
"docker_app_image=httpd docker_app_name=apache"

 Old container replaced automatically
 Nginx still works (no changes needed)

 16. WHAT YOU JUST BUILT

What we know:

Inventory
Variables + Vault
Roles
Templates
Docker automation
Reverse proxy setup
Idempotent deployments
Production-style architecture