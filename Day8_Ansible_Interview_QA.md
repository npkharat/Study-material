# ⚙️ DevOps Interview Preparation — Day 8: Ansible
### For 3 Years Experience | 110+ Questions & Answers

> **Tip:** Use `Ctrl+Shift+V` in VS Code to preview | Push to GitHub for auto-rendering
> **Previous:** Day 7 - Git | **Next:** Day 9 - Monitoring

---

## 📋 Table of Contents
1. [Core Concepts (Q1–Q20)](#1-core-concepts)
2. [Inventory & Variables (Q21–Q35)](#2-inventory--variables)
3. [Playbooks & Tasks (Q36–Q55)](#3-playbooks--tasks)
4. [Roles & Collections (Q56–Q68)](#4-roles--collections)
5. [Ansible Vault & Security (Q69–Q78)](#5-ansible-vault--security)
6. [Advanced & Real-World (Q79–Q110)](#6-advanced--real-world)

---

## 1. Core Concepts

**Q1. What is Ansible?**
> Ansible is an open-source IT automation tool used for:
> - **Configuration management** — install software, configure servers
> - **Application deployment** — deploy apps consistently
> - **Orchestration** — coordinate complex multi-step workflows
> - **Provisioning** — set up infrastructure
> Written in Python. Uses YAML for playbooks. Agentless — no software needed on managed nodes.

---

**Q2. What makes Ansible different from other tools like Chef/Puppet?**
> | Feature | Ansible | Chef | Puppet |
> |---|---|---|---|
> | Language | YAML | Ruby DSL | Puppet DSL |
> | Architecture | Agentless (SSH) | Agent-based | Agent-based |
> | Learning curve | Easy | Steep | Moderate |
> | Push/Pull | Push | Pull | Pull |
> | Setup | Minimal | Complex | Complex |
> Ansible's agentless nature means zero setup on managed hosts — just SSH and Python.

---

**Q3. What is Agentless architecture in Ansible?**
> Ansible doesn't require any agent/daemon installed on managed nodes. It:
> - Connects via **SSH** (Linux) or **WinRM** (Windows)
> - Copies Python scripts temporarily
> - Executes them
> - Removes them
> Benefits: No agent maintenance, no port management for agents, works on any Python-capable server.

---

**Q4. What is a Control Node and Managed Node?**
> - **Control Node:** The machine where Ansible is installed and runs from. Can be your laptop or a CI/CD server. Must have Python and Ansible installed.
> - **Managed Node (Host):** The servers being configured. Only needs SSH access and Python (2.7+ or 3.5+).

---

**Q5. What is an Ansible Playbook?**
> A Playbook is a YAML file defining what Ansible should do — ordered list of plays, each targeting specific hosts and running tasks.
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

---

**Q6. What is an Ansible Module?**
> A module is a reusable unit of code that performs a specific task. Ansible has 3000+ built-in modules.
> Common modules:
> - `apt` / `yum` / `dnf` — package management
> - `copy` / `template` — file management
> - `service` / `systemd` — service management
> - `user` / `group` — user management
> - `command` / `shell` — run commands
> - `file` — file/directory management
> - `git` — Git operations
> - `docker_container` — Docker management

---

**Q7. What is the difference between `command` and `shell` module?**
> - **`command`:** Runs command directly. No shell features (pipes, redirects, variables). Safer.
> - **`shell`:** Runs through `/bin/sh`. Supports pipes `|`, redirects `>`, variables `$VAR`. Less safe.
```yaml
- name: Simple command (no shell needed)
  command: systemctl restart nginx

- name: Command with pipe (needs shell)
  shell: ps aux | grep nginx | wc -l

- name: Command with redirect
  shell: echo "hello" > /tmp/test.txt
```
> Best practice: Use `command` when possible, `shell` only when you need shell features.

---

**Q8. What is Idempotency in Ansible?**
> Running the same playbook multiple times should produce the same result without making unnecessary changes.
> - If nginx is already installed → don't reinstall
> - If user already exists → don't create again
> - If file already has correct content → don't rewrite
> Most Ansible modules are idempotent by design. `command`/`shell` are NOT idempotent — use `creates`/`removes` flags or `changed_when`.

---

**Q9. What is `ansible-playbook` command?**
```bash
ansible-playbook playbook.yml                    # run playbook
ansible-playbook playbook.yml -i inventory.ini  # specify inventory
ansible-playbook playbook.yml --check           # dry run (no changes)
ansible-playbook playbook.yml --diff            # show file diffs
ansible-playbook playbook.yml -v                # verbose
ansible-playbook playbook.yml -vvv              # very verbose (debug)
ansible-playbook playbook.yml --tags "install"  # run only tagged tasks
ansible-playbook playbook.yml --skip-tags "restart"
ansible-playbook playbook.yml --limit "web01"   # run on specific host
ansible-playbook playbook.yml -e "env=prod"     # extra variables
ansible-playbook playbook.yml --start-at-task "Install packages"
```

---

**Q10. What is `ansible` ad-hoc command?**
> Run a single task without writing a playbook:
```bash
ansible all -m ping                              # ping all hosts
ansible webservers -m apt -a "name=nginx state=present" -b
ansible all -m shell -a "df -h"                 # check disk space
ansible db -m service -a "name=mysql state=restarted" -b
ansible all -m copy -a "src=file.txt dest=/tmp/file.txt"
ansible all -m setup                            # gather facts
ansible webservers -m command -a "uptime"

# Flags:
# -m = module
# -a = arguments
# -b = become (sudo)
# -i = inventory file
# -u = remote user
# -k = ask SSH password
# --private-key = SSH key file
```

---

**Q11. What is an Ansible Inventory?**
> Inventory defines the hosts (servers) Ansible manages. Can be static (file) or dynamic (script/plugin).
```ini
# Static inventory (ini format)
[webservers]
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11

[dbservers]
db01 ansible_host=192.168.1.20

[production:children]
webservers
dbservers
```

---

**Q12. What is `ansible.cfg` file?**
> Configuration file for Ansible settings. Ansible looks for it in this order:
> 1. `ANSIBLE_CONFIG` environment variable
> 2. `./ansible.cfg` (current directory)
> 3. `~/.ansible.cfg` (user home)
> 4. `/etc/ansible/ansible.cfg` (global)
```ini
[defaults]
inventory = ./inventory
remote_user = ubuntu
private_key_file = ~/.ssh/id_rsa
host_key_checking = False
retry_files_enabled = False
stdout_callback = yaml
forks = 20

[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

---

**Q13. What is `gather_facts` in Ansible?**
> When a playbook runs, Ansible first collects information about the managed host (called facts):
> - OS type, version
> - IP addresses
> - Memory, CPU info
> - Hostname
> - Disk info
```yaml
- hosts: all
  gather_facts: yes  # default
  tasks:
    - name: Print OS
      debug:
        msg: "OS is {{ ansible_distribution }} {{ ansible_distribution_version }}"

# Disable for faster execution if facts not needed
- hosts: all
  gather_facts: no
```

---

**Q14. What are Ansible Facts?**
> Facts are system properties collected by the `setup` module automatically.
```bash
ansible web01 -m setup                          # see all facts
ansible web01 -m setup -a "filter=ansible_*ip*" # filter facts
```
> Common facts:
```yaml
ansible_hostname          # server hostname
ansible_os_family         # RedHat, Debian, Windows
ansible_distribution      # Ubuntu, CentOS, Amazon
ansible_memtotal_mb       # total memory in MB
ansible_processor_vcpus   # number of CPUs
ansible_default_ipv4.address  # primary IP
ansible_env.HOME          # home directory
```

---

**Q15. What is `become` in Ansible?**
> `become` enables privilege escalation (running as another user, typically root):
```yaml
- hosts: webservers
  become: yes           # become root for all tasks
  become_user: root     # which user to become (default: root)
  become_method: sudo   # how to become (sudo, su, pbrun, pfexec)

  tasks:
    - name: Install package (needs root)
      apt:
        name: nginx
        state: present

    - name: Write to /tmp (no root needed)
      copy:
        content: "hello"
        dest: /tmp/test.txt
      become: no        # override for specific task
```

---

**Q16. What is the `debug` module?**
```yaml
- name: Print variable value
  debug:
    msg: "Server IP is {{ ansible_host }}"

- name: Print variable (simple)
  debug:
    var: ansible_distribution

- name: Print with verbosity (only show with -v)
  debug:
    msg: "Detailed info: {{ some_var }}"
    verbosity: 2
```

---

**Q17. What is `register` in Ansible?**
> Captures the output of a task to use in subsequent tasks:
```yaml
- name: Check if file exists
  stat:
    path: /etc/myapp/config.yml
  register: config_file

- name: Create config if not exists
  copy:
    content: "default config"
    dest: /etc/myapp/config.yml
  when: not config_file.stat.exists

- name: Print command output
  command: cat /etc/os-release
  register: os_release

- debug:
    msg: "{{ os_release.stdout }}"
```

---

**Q18. What is `when` in Ansible?**
> Conditional execution — run a task only if condition is true:
```yaml
- name: Install on Ubuntu only
  apt:
    name: nginx
    state: present
  when: ansible_distribution == "Ubuntu"

- name: Install on CentOS only
  yum:
    name: nginx
    state: present
  when: ansible_distribution == "CentOS"

- name: Run only if variable is set
  command: deploy.sh
  when: deploy_version is defined

- name: Multiple conditions
  service:
    name: nginx
    state: restarted
  when:
    - ansible_os_family == "Debian"
    - nginx_config.changed
    - env == "production"
```

---

**Q19. What is `notify` and `handlers`?**
> Handlers are tasks that run ONLY when notified AND only once at the end of the play:
```yaml
tasks:
  - name: Copy nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx       # notify handler

  - name: Copy SSL certificate
    copy:
      src: cert.pem
      dest: /etc/nginx/cert.pem
    notify: Restart nginx       # same handler - runs only once!

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```
> If config or cert changes, nginx restarts once (not twice). If nothing changes, handler doesn't run.

---

**Q20. What is `ansible-lint`?**
```bash
ansible-lint playbook.yml       # check for issues
ansible-lint roles/             # check role
ansible-lint                    # check everything
```
> Checks playbooks and roles for best practices, style issues, potential bugs. Use in CI/CD to enforce quality.

---

## 2. Inventory & Variables

**Q21. Write a YAML format inventory.**
```yaml
# inventory.yml
all:
  children:
    production:
      children:
        webservers:
          hosts:
            web01:
              ansible_host: 10.0.1.10
              nginx_port: 80
            web02:
              ansible_host: 10.0.1.11
              nginx_port: 80
        dbservers:
          hosts:
            db01:
              ansible_host: 10.0.1.20
              db_name: myapp
    staging:
      hosts:
        stage01:
          ansible_host: 10.0.2.10
  vars:
    ansible_user: ubuntu
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

---

**Q22. What are host variables and group variables?**
```
inventory/
├── hosts.yml          # main inventory
├── host_vars/
│   ├── web01.yml      # variables for web01 only
│   └── db01.yml       # variables for db01 only
└── group_vars/
    ├── all.yml        # variables for ALL hosts
    ├── webservers.yml # variables for webservers group
    └── production.yml # variables for production group
```
```yaml
# group_vars/webservers.yml
nginx_worker_processes: 4
nginx_max_connections: 1024
app_port: 8080

# group_vars/all.yml
ansible_user: ubuntu
ntp_server: pool.ntp.org
timezone: Asia/Kolkata
```

---

**Q23. What is variable precedence in Ansible?**
> Variables have priority (lowest to highest):
> 1. Role defaults (`roles/x/defaults/main.yml`)
> 2. Inventory group_vars/all
> 3. Inventory group_vars/<group>
> 4. Inventory host_vars/<host>
> 5. Playbook group_vars/all
> 6. Playbook group_vars/<group>
> 7. Playbook host_vars/<host>
> 8. Host facts
> 9. Play vars
> 10. Task vars
> 11. `extra_vars` (`-e`) — **HIGHEST PRIORITY**

---

**Q24. How do you define variables in a playbook?**
```yaml
- hosts: webservers
  vars:
    nginx_port: 80
    app_name: myapp
    packages:
      - nginx
      - curl
      - git

  vars_files:
    - vars/common.yml
    - vars/{{ env }}.yml     # load different vars per env

  tasks:
    - name: Install packages
      apt:
        name: "{{ packages }}"
        state: present
```

---

**Q25. What is `set_fact` module?**
```yaml
- name: Set dynamic variable
  set_fact:
    app_version: "{{ lookup('file', 'VERSION') }}"
    deploy_timestamp: "{{ ansible_date_time.iso8601 }}"
    is_primary: "{{ inventory_hostname == groups['dbservers'][0] }}"

- name: Use the fact
  debug:
    msg: "Deploying version {{ app_version }}"
```
> Creates variables dynamically during playbook execution. Useful for computed values.

---

**Q26. What are magic variables in Ansible?**
```yaml
{{ inventory_hostname }}       # current host's name in inventory
{{ inventory_hostname_short }} # short hostname (no domain)
{{ group_names }}              # list of groups current host belongs to
{{ groups }}                   # dict of all groups and their hosts
{{ groups['webservers'] }}     # list of hosts in webservers group
{{ hostvars }}                 # variables of all hosts
{{ hostvars['web01']['ansible_host'] }} # specific host's variable
{{ play_hosts }}               # hosts in current play
{{ ansible_play_batch }}       # current batch of hosts
```

---

**Q27. What is a dynamic inventory?**
> Instead of a static file, inventory is generated dynamically from cloud APIs:
```bash
# AWS dynamic inventory
ansible-inventory -i aws_ec2.yml --list
ansible-playbook -i aws_ec2.yml playbook.yml
```
```yaml
# aws_ec2.yml
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1
filters:
  tag:Environment: production
keyed_groups:
  - key: tags.Role
    prefix: role
compose:
  ansible_host: public_ip_address
```
> Automatically discovers EC2 instances tagged with `Environment=production`.

---

**Q28. How do you use `vars_prompt` to ask for input?**
```yaml
- hosts: all
  vars_prompt:
    - name: deploy_version
      prompt: "Enter version to deploy"
      private: no

    - name: db_password
      prompt: "Enter database password"
      private: yes           # don't echo to terminal

  tasks:
    - name: Deploy version
      debug:
        msg: "Deploying {{ deploy_version }}"
```

---

**Q29. What is `lookup` in Ansible?**
```yaml
# Read from file
- debug:
    msg: "{{ lookup('file', '/etc/hostname') }}"

# Read environment variable
- debug:
    msg: "{{ lookup('env', 'HOME') }}"

# Read from CSV
- debug:
    msg: "{{ lookup('csvfile', 'alice file=users.csv col=1') }}"

# Random password
- debug:
    msg: "{{ lookup('password', '/tmp/pass length=16 chars=ascii_letters,digits') }}"

# URL content
- debug:
    msg: "{{ lookup('url', 'http://api.example.com/version') }}"
```

---

**Q30. How do you use Jinja2 filters in Ansible?**
```yaml
# String filters
{{ var | upper }}              # UPPERCASE
{{ var | lower }}              # lowercase
{{ var | capitalize }}         # Capitalize
{{ var | replace('a', 'b') }} # replace
{{ var | trim }}               # remove whitespace
{{ var | default('fallback') }} # use fallback if var is undefined

# List/dict filters
{{ list | length }}            # count items
{{ list | join(', ') }}        # join list to string
{{ list | sort }}              # sort list
{{ list | unique }}            # remove duplicates
{{ dict | dict2items }}        # convert dict to list
{{ items | items2dict }}       # convert list to dict

# Type filters
{{ var | int }}                # convert to integer
{{ var | string }}             # convert to string
{{ var | bool }}               # convert to boolean
{{ var | to_json }}            # convert to JSON
{{ var | from_json }}          # parse JSON

# Useful DevOps filters
{{ path | basename }}          # /etc/nginx/nginx.conf → nginx.conf
{{ path | dirname }}           # /etc/nginx/nginx.conf → /etc/nginx
{{ var | b64encode }}          # base64 encode
{{ var | b64decode }}          # base64 decode
{{ var | hash('sha1') }}       # hash value
```

---

**Q31. What is `with_items` / `loop`?**
```yaml
# Modern way: loop
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - git
    - htop

# Loop over dict
- name: Create users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
    state: present
  loop:
    - { name: alice, groups: sudo }
    - { name: bob, groups: docker }

# Loop with index
- name: Numbered task
  debug:
    msg: "Item {{ loop.index }}: {{ item }}"
  loop: [a, b, c]

# Old way (still works)
- apt:
    name: "{{ item }}"
  with_items:
    - nginx
    - curl
```

---

**Q32. What is `until` loop in Ansible?**
```yaml
- name: Wait for application to start
  uri:
    url: http://localhost:8080/health
    status_code: 200
  register: result
  until: result.status == 200
  retries: 10
  delay: 5
  # Retries 10 times with 5 second delay until HTTP 200
```

---

**Q33. What is `include_vars`?**
```yaml
- name: Load environment-specific variables
  include_vars:
    file: "vars/{{ env }}.yml"

- name: Load multiple var files
  include_vars:
    dir: vars/
    extensions: [yml, yaml]
```

---

**Q34. What is a `block` in Ansible?**
```yaml
- name: Install and configure app block
  block:
    - name: Install app
      apt:
        name: myapp
        state: present

    - name: Configure app
      template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf

    - name: Start app
      service:
        name: myapp
        state: started

  rescue:
    - name: Handle failure
      debug:
        msg: "Installation failed, rolling back..."
    - name: Remove partial install
      apt:
        name: myapp
        state: absent

  always:
    - name: Log attempt
      shell: echo "Install attempted at $(date)" >> /var/log/deploys.log
```

---

**Q35. What is `delegate_to`?**
```yaml
- name: Remove server from load balancer
  command: lb-cli remove {{ inventory_hostname }}
  delegate_to: loadbalancer.example.com

- name: Run task on localhost
  command: notify-slack.py "Deploying to {{ inventory_hostname }}"
  delegate_to: localhost

- name: Create DB backup before deploy
  mysql_db:
    name: myapp
    state: dump
    target: /backup/myapp.sql
  delegate_to: db01
```

---

## 3. Playbooks & Tasks

**Q36. Write a complete playbook to install and configure Nginx.**
```yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes

  vars:
    nginx_port: 80
    server_name: myapp.example.com
    doc_root: /var/www/myapp

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Create document root
      file:
        path: "{{ doc_root }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'

    - name: Deploy Nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/myapp
        owner: root
        group: root
        mode: '0644'
      notify: Reload Nginx

    - name: Enable site
      file:
        src: /etc/nginx/sites-available/myapp
        dest: /etc/nginx/sites-enabled/myapp
        state: link
      notify: Reload Nginx

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```

---

**Q37. What is the `template` module?**
> Copies a Jinja2 template file to a managed host, replacing variables:
```jinja2
{# templates/nginx.conf.j2 #}
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};

    root {{ doc_root }};
    index index.html;

    {% if ssl_enabled %}
    ssl_certificate /etc/nginx/cert.pem;
    ssl_certificate_key /etc/nginx/key.pem;
    {% endif %}

    access_log /var/log/nginx/{{ server_name }}.access.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

**Q38. What is the `copy` module?**
```yaml
# Copy file from control node to managed host
- name: Copy config file
  copy:
    src: files/app.conf
    dest: /etc/myapp/app.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes        # backup existing file

# Copy inline content
- name: Create file with content
  copy:
    content: |
      DB_HOST=localhost
      DB_PORT=5432
      DB_NAME=myapp
    dest: /etc/myapp/.env
    mode: '0600'
```

---

**Q39. What is the `file` module?**
```yaml
# Create directory
- file:
    path: /opt/myapp/logs
    state: directory
    owner: appuser
    group: appuser
    mode: '0755'

# Create symlink
- file:
    src: /opt/myapp/current
    dest: /opt/myapp/releases/v1.0.0
    state: link

# Delete file
- file:
    path: /tmp/old-file.txt
    state: absent

# Touch file (create if not exists)
- file:
    path: /var/log/myapp.log
    state: touch
    modification_time: preserve
    access_time: preserve
```

---

**Q40. What is the `lineinfile` module?**
```yaml
# Ensure a line exists in a file
- lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'

# Add line after a pattern
- lineinfile:
    path: /etc/hosts
    insertafter: '^127.0.0.1'
    line: '10.0.0.10 db-server'

# Remove a line
- lineinfile:
    path: /etc/hosts
    regexp: '^10.0.0.9 old-server'
    state: absent
```

---

**Q41. What is the `blockinfile` module?**
```yaml
# Insert a block of text in a file
- blockinfile:
    path: /etc/nginx/nginx.conf
    block: |
      upstream myapp {
          server 10.0.1.10:8080;
          server 10.0.1.11:8080;
      }
    marker: "# {mark} ANSIBLE MANAGED BLOCK"
    insertbefore: "http {"
```

---

**Q42. What is the `user` module?**
```yaml
- name: Create application user
  user:
    name: appuser
    uid: 1500
    group: appgroup
    groups:
      - docker
      - sudo
    shell: /bin/bash
    home: /home/appuser
    create_home: yes
    password: "{{ 'mypassword' | password_hash('sha512') }}"
    comment: "Application User"
    state: present

- name: Add SSH key for user
  authorized_key:
    user: appuser
    state: present
    key: "{{ lookup('file', 'files/appuser.pub') }}"
```

---

**Q43. What are `pre_tasks` and `post_tasks`?**
```yaml
- hosts: webservers
  become: yes

  pre_tasks:
    - name: Remove from load balancer
      command: lb-cli remove {{ inventory_hostname }}
      delegate_to: localhost

    - name: Wait for connections to drain
      wait_for:
        timeout: 30

  roles:
    - nginx
    - myapp

  tasks:
    - name: Verify application
      uri:
        url: http://localhost:8080/health
        status_code: 200

  post_tasks:
    - name: Add back to load balancer
      command: lb-cli add {{ inventory_hostname }}
      delegate_to: localhost
```

---

**Q44. What is `ignore_errors` and `failed_when`?**
```yaml
- name: Try to stop service (may not exist)
  service:
    name: myapp
    state: stopped
  ignore_errors: yes      # continue even if this fails

- name: Run command, define when it's considered failed
  command: check-health.sh
  register: health_result
  failed_when:
    - health_result.rc != 0
    - "'CRITICAL' in health_result.stdout"
  # Fail only if exit code != 0 AND output contains CRITICAL
```

---

**Q45. What is `changed_when`?**
```yaml
- name: Check if reboot needed
  command: needs-restarting -r
  register: reboot_check
  changed_when: reboot_check.rc == 1   # define when task is "changed"
  failed_when: reboot_check.rc > 1     # define when it's "failed"

- name: Import data (never shows changed)
  command: import-data.sh
  changed_when: false    # always show as ok, never changed
```

---

**Q46. What is `tags` in Ansible?**
```yaml
tasks:
  - name: Install packages
    apt:
      name: nginx
      state: present
    tags:
      - install
      - packages

  - name: Configure nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    tags:
      - configure
      - nginx

  - name: Restart nginx
    service:
      name: nginx
      state: restarted
    tags:
      - restart
      - never      # special tag - only runs when explicitly specified
```
```bash
ansible-playbook playbook.yml --tags "install,configure"
ansible-playbook playbook.yml --skip-tags "restart"
ansible-playbook playbook.yml --tags "never,restart"  # run 'never' tasks
```

---

**Q47. What is `serial` in Ansible?**
```yaml
- hosts: webservers
  serial: 1          # deploy to 1 server at a time (rolling deploy)
  # serial: 2        # 2 at a time
  # serial: "25%"    # 25% of hosts at a time

  tasks:
    - name: Deploy app
      # ... deployment tasks
```
> For rolling deployments — update servers one by one to avoid downtime.

---

**Q48. What is `max_fail_percentage`?**
```yaml
- hosts: webservers
  serial: "25%"
  max_fail_percentage: 20   # abort if more than 20% of hosts fail

  tasks:
    - name: Deploy
      # ... if too many hosts fail, stop entire play
```

---

**Q49. What is `any_errors_fatal`?**
```yaml
- hosts: webservers
  any_errors_fatal: true    # stop ALL hosts if ANY host fails

  tasks:
    - name: Critical step
      command: critical-migrate.sh
```
> Default behavior: if web01 fails, Ansible continues on web02, web03, etc. With `any_errors_fatal: true`, one failure stops everything.

---

**Q50. What is the `wait_for` module?**
```yaml
# Wait for port to be open
- name: Wait for application to start
  wait_for:
    host: localhost
    port: 8080
    delay: 5        # wait 5s before first check
    timeout: 60     # fail if not up in 60s
    state: started

# Wait for file to exist
- wait_for:
    path: /tmp/app.ready
    state: present
    timeout: 120

# Wait for port to close (during shutdown)
- wait_for:
    port: 8080
    state: stopped
    timeout: 30
```

---

**Q51. What is `include_tasks` and `import_tasks`?**
```yaml
# import_tasks: static (processed at parse time)
- import_tasks: tasks/install.yml

# include_tasks: dynamic (processed at runtime, can use variables)
- include_tasks: "tasks/{{ ansible_os_family }}.yml"

- include_tasks: tasks/deploy.yml
  vars:
    version: "1.0.0"
  when: env == "production"
```
> Key difference: `import_tasks` is static (tags apply to included tasks), `include_tasks` is dynamic (can use variables in filename).

---

**Q52. What is `run_once`?**
```yaml
- name: Create database schema (run on first host only)
  command: python manage.py migrate
  run_once: true           # runs on FIRST host in play only

- name: Notify deployment (run once on specific host)
  command: notify-slack.sh
  run_once: true
  delegate_to: "{{ groups['webservers'][0] }}"
```

---

**Q53. What is `async` and `poll` in Ansible?**
```yaml
# Start long-running task in background
- name: Run long backup
  command: /backup.sh
  async: 3600      # allow up to 1 hour
  poll: 0          # don't wait, continue
  register: backup_job

# Check job later
- name: Wait for backup to complete
  async_status:
    jid: "{{ backup_job.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 60
  delay: 60
```

---

**Q54. What is the `uri` module?**
```yaml
- name: Call REST API
  uri:
    url: https://api.example.com/deploy
    method: POST
    body_format: json
    body:
      version: "{{ version }}"
      environment: "{{ env }}"
    headers:
      Authorization: "Bearer {{ api_token }}"
      Content-Type: application/json
    status_code: [200, 201]
  register: api_response

- name: Health check
  uri:
    url: http://localhost:8080/health
    status_code: 200
    return_content: yes
  register: health
  until: health.status == 200
  retries: 10
  delay: 5
```

---

**Q55. What is the `assert` module?**
```yaml
- name: Verify deployment succeeded
  assert:
    that:
      - app_version == expected_version
      - health_check.status == 200
      - disk_space.stdout | int > 1000
    fail_msg: "Deployment verification failed!"
    success_msg: "All checks passed!"
```

---

## 4. Roles & Collections

**Q56. What is an Ansible Role?**
> A role is a structured way to organize playbook content. It promotes reusability.
```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml      # main task list
    ├── handlers/
    │   └── main.yml      # handlers
    ├── templates/
    │   └── nginx.conf.j2 # Jinja2 templates
    ├── files/
    │   └── cert.pem      # static files
    ├── vars/
    │   └── main.yml      # role variables (high priority)
    ├── defaults/
    │   └── main.yml      # default variables (lowest priority)
    ├── meta/
    │   └── main.yml      # role metadata, dependencies
    └── README.md
```

---

**Q57. How do you create and use a role?**
```bash
# Create role structure automatically
ansible-galaxy init roles/nginx
```
```yaml
# Use role in playbook
- hosts: webservers
  roles:
    - nginx
    - { role: myapp, app_port: 8080 }
    - { role: monitoring, when: env == "production" }
```

---

**Q58. What is `defaults/main.yml` vs `vars/main.yml` in a role?**
> - **`defaults/main.yml`:** Lowest priority variables. Easily overridden by anything. Use for "sensible defaults".
>   ```yaml
>   nginx_port: 80
>   nginx_worker_processes: auto
>   ```
> - **`vars/main.yml`:** Higher priority. Hard to override (only extra_vars beat it). Use for "role internals that shouldn't be changed".
>   ```yaml
>   nginx_config_dir: /etc/nginx
>   nginx_log_dir: /var/log/nginx
>   ```

---

**Q59. What are role dependencies?**
```yaml
# roles/myapp/meta/main.yml
dependencies:
  - role: nginx
    vars:
      nginx_port: 8080
  - role: java
    vars:
      java_version: "17"
  - role: common
```
> When `myapp` role is used, Ansible automatically runs `common`, `java`, and `nginx` first.

---

**Q60. What is Ansible Galaxy?**
> Ansible Galaxy is a public repository of Ansible roles and collections shared by the community.
```bash
ansible-galaxy install geerlingguy.nginx      # install role
ansible-galaxy install -r requirements.yml    # install from requirements file
ansible-galaxy list                            # list installed roles
ansible-galaxy search nginx                   # search roles
ansible-galaxy remove geerlingguy.nginx       # remove role
```
```yaml
# requirements.yml
roles:
  - name: geerlingguy.nginx
    version: 3.0.0
  - name: geerlingguy.java
    version: 2.0.0

collections:
  - name: community.general
    version: ">=5.0"
  - name: amazon.aws
```

---

**Q61. What is an Ansible Collection?**
> Collections are a distribution format for Ansible content — modules, plugins, roles, and playbooks packaged together.
```bash
ansible-galaxy collection install amazon.aws
ansible-galaxy collection install community.general
ansible-galaxy collection install -r requirements.yml
```
```yaml
# Use collection module
- name: Create S3 bucket
  amazon.aws.s3_bucket:
    name: my-bucket
    state: present
    region: ap-south-1
```

---

**Q62. How do you test an Ansible role?**
> Use **Molecule** — testing framework for Ansible roles:
```bash
pip install molecule molecule-docker

molecule init scenario            # create test scenario
molecule test                     # full test (create, converge, verify, destroy)
molecule converge                 # apply role to test instance
molecule verify                   # run assertions
molecule destroy                  # remove test instance
molecule login                    # SSH into test instance
```
```yaml
# molecule/default/verify.yml
- name: Verify
  hosts: all
  tasks:
    - name: Check nginx is running
      service_facts:
    - assert:
        that: "'nginx' in services and services['nginx'].state == 'running'"
```

---

**Q63. What is the `include_role` vs `import_role`?**
```yaml
# import_role: static (at parse time)
- import_role:
    name: nginx

# include_role: dynamic (at runtime)
- include_role:
    name: "{{ role_name }}"     # can use variables in name
  vars:
    role_var: value
  when: install_nginx
```

---

**Q64. How do you share roles across projects?**
> Options:
> 1. **Ansible Galaxy** — publish to public galaxy
> 2. **Private Galaxy** — Ansible Automation Platform
> 3. **Git submodules** — include role repo as submodule
> 4. **requirements.yml** — specify Git URL for role
```yaml
# requirements.yml
roles:
  - name: nginx-role
    src: https://github.com/myorg/ansible-nginx-role.git
    version: v2.0.0
    scm: git
```

---

**Q65. What is a playbook vs role vs task vs collection?**
> - **Task:** Single unit of work (install nginx, create user)
> - **Playbook:** Orchestrates tasks against hosts (YAML file)
> - **Role:** Organized, reusable collection of tasks/templates/files
> - **Collection:** Package of roles, modules, plugins published together

---

**Q66. What is a `custom module` in Ansible?**
> You can write your own modules in Python:
```python
# library/my_module.py
from ansible.module_utils.basic import AnsibleModule

def main():
    module = AnsibleModule(argument_spec=dict(
        name=dict(type='str', required=True),
        state=dict(type='str', default='present', choices=['present', 'absent']),
    ))
    name = module.params['name']
    # ... your logic ...
    module.exit_json(changed=True, message=f"Created {name}")

if __name__ == '__main__':
    main()
```

---

**Q67. What is an Ansible callback plugin?**
> Callback plugins respond to Ansible events — customize output format, send notifications:
```ini
# ansible.cfg
[defaults]
stdout_callback = yaml          # pretty YAML output
# stdout_callback = json        # JSON output (for CI/CD)
# stdout_callback = minimal     # minimal output
callback_whitelist = timer, profile_tasks
```
> Popular callbacks: `timer` (shows run time), `profile_tasks` (shows per-task timing), `slack` (post results to Slack).

---

**Q68. How do you write a custom filter plugin?**
```python
# filter_plugins/custom_filters.py
def to_app_port(service_name):
    ports = {'web': 80, 'api': 8080, 'db': 5432}
    return ports.get(service_name, 8000)

class FilterModule(object):
    def filters(self):
        return {'to_app_port': to_app_port}
```
```yaml
# Usage in playbook
- debug:
    msg: "Port is {{ 'web' | to_app_port }}"
```

---

## 5. Ansible Vault & Security

**Q69. What is Ansible Vault?**
> Ansible Vault encrypts sensitive data (passwords, keys, certificates) so they can safely be stored in version control.
```bash
# Encrypt a file
ansible-vault encrypt secrets.yml

# Decrypt a file
ansible-vault decrypt secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Encrypt a string
ansible-vault encrypt_string 'mypassword' --name 'db_password'

# Create new encrypted file
ansible-vault create secrets.yml

# Re-key (change vault password)
ansible-vault rekey secrets.yml
```

---

**Q70. How do you use a vault-encrypted file in a playbook?**
```yaml
# vars/secrets.yml (encrypted with vault)
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  61383132303163613266323565353...

# In playbook
- hosts: all
  vars_files:
    - vars/secrets.yml
  tasks:
    - name: Configure database
      template:
        src: db.conf.j2
        dest: /etc/app/db.conf
```
```bash
# Run playbook with vault password
ansible-playbook playbook.yml --ask-vault-pass
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

---

**Q71. What is a vault ID?**
```bash
# Encrypt with specific vault ID
ansible-vault encrypt_string 'secret' --vault-id prod@~/.prod_vault_pass --name 'api_key'

# Decrypt - provide multiple vault IDs
ansible-playbook playbook.yml \
  --vault-id dev@~/.dev_vault_pass \
  --vault-id prod@~/.prod_vault_pass
```
> Vault IDs allow different passwords for different environments (dev vault, prod vault).

---

**Q72. How do you secure SSH in Ansible?**
```yaml
# Use SSH keys (never passwords in automation)
ansible_ssh_private_key_file: ~/.ssh/id_rsa

# Disable host key checking (dev only - not prod!)
host_key_checking = False  # in ansible.cfg

# Use jump host / bastion
ansible_ssh_common_args: '-o ProxyJump=bastion.example.com'
```

---

**Q73. How do you avoid logging sensitive data?**
```yaml
- name: Set database password
  command: set-password.sh {{ db_password }}
  no_log: true          # suppress task output in logs

- name: Debug non-sensitive info
  debug:
    msg: "Deployed version {{ version }}"
  # This IS logged
```

---

**Q74. What is SSH agent forwarding in Ansible?**
```ini
# ansible.cfg
[ssh_connection]
ssh_args = -o ForwardAgent=yes
```
> Allows Ansible to use your local SSH keys when connecting to further hosts from the managed node (like Git cloning from a private repo on the managed host).

---

**Q75. How do you use `become` securely?**
```yaml
# Only escalate when necessary
- name: Read config (no root needed)
  command: cat /etc/myapp/config.yml
  # No become here

- name: Install package (root needed)
  apt:
    name: nginx
    state: present
  become: yes
  become_user: root
```
```ini
# Sudo without password for specific commands (in /etc/sudoers)
appuser ALL=(ALL) NOPASSWD: /usr/bin/apt, /bin/systemctl
```

---

**Q76. What is `ansible-runner`?**
> Ansible Runner is a Python library and CLI for executing Ansible playbooks in a controlled, reproducible way. Used by AWX and Ansible Automation Platform internally.
```bash
ansible-runner run /path/to/project -p playbook.yml
```

---

**Q77. What is AWX / Ansible Automation Platform?**
> AWX is the open-source web UI for Ansible (upstream of Red Hat's Ansible Automation Platform).
> Features:
> - Web interface for running playbooks
> - Role-based access control (RBAC)
> - Inventory management
> - Credential storage
> - Scheduled jobs
> - REST API
> - Workflow templates (chain multiple playbooks)

---

**Q78. What is best practice for Ansible project structure?**
```
ansible-project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   └── webservers.yml
│   │   └── host_vars/
│   │       └── web01.yml
│   └── staging/
│       └── hosts.yml
├── playbooks/
│   ├── site.yml          # master playbook
│   ├── webservers.yml
│   └── databases.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── myapp/
├── collections/
│   └── requirements.yml
└── vault/
    └── secrets.yml       # encrypted
```

---

## 6. Advanced & Real-World

**Q79. Write a rolling deployment playbook.**
```yaml
---
- name: Rolling deployment
  hosts: webservers
  serial: 1
  become: yes

  vars:
    app_version: "{{ version | default('latest') }}"
    app_dir: /opt/myapp

  pre_tasks:
    - name: Remove from load balancer
      uri:
        url: "http://{{ lb_host }}/remove/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost

    - name: Wait for active connections to drain
      wait_for:
        timeout: 15

  tasks:
    - name: Pull new Docker image
      docker_image:
        name: "myrepo/myapp:{{ app_version }}"
        source: pull

    - name: Stop old container
      docker_container:
        name: myapp
        state: stopped

    - name: Start new container
      docker_container:
        name: myapp
        image: "myrepo/myapp:{{ app_version }}"
        state: started
        ports:
          - "8080:8080"
        restart_policy: unless-stopped

    - name: Wait for app to be healthy
      uri:
        url: http://localhost:8080/health
        status_code: 200
      register: health
      until: health.status == 200
      retries: 12
      delay: 5

  post_tasks:
    - name: Add back to load balancer
      uri:
        url: "http://{{ lb_host }}/add/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost
```

---

**Q80. How do you use Ansible with Docker?**
```yaml
- name: Manage Docker containers
  hosts: dockerhosts
  become: yes

  tasks:
    - name: Pull image
      docker_image:
        name: nginx:latest
        source: pull

    - name: Run container
      docker_container:
        name: webserver
        image: nginx:latest
        state: started
        ports:
          - "80:80"
          - "443:443"
        volumes:
          - /etc/nginx/conf.d:/etc/nginx/conf.d:ro
        env:
          NGINX_HOST: myapp.example.com
        restart_policy: always

    - name: Run docker-compose
      community.docker.docker_compose:
        project_src: /opt/myapp
        state: present
```

---

**Q81. How do you use Ansible with Kubernetes?**
```yaml
- name: Deploy to Kubernetes
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Deploy application
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: myapp
            namespace: production
          spec:
            replicas: 3
            selector:
              matchLabels:
                app: myapp
            template:
              metadata:
                labels:
                  app: myapp
              spec:
                containers:
                - name: myapp
                  image: "myrepo/myapp:{{ version }}"

    - name: Wait for rollout
      kubernetes.core.k8s_rollout_status:
        name: myapp
        namespace: production
        kind: Deployment
```

---

**Q82. How do you use Ansible with AWS?**
```yaml
- name: Provision AWS infrastructure
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Create EC2 instance
      amazon.aws.ec2_instance:
        name: "web-{{ env }}"
        key_name: my-keypair
        instance_type: t3.micro
        image_id: ami-0abcdef1234567890
        security_groups:
          - web-sg
        subnet_id: subnet-12345
        tags:
          Environment: "{{ env }}"
          Role: webserver
      register: ec2_result

    - name: Add new instance to inventory
      add_host:
        hostname: "{{ ec2_result.instances[0].public_ip_address }}"
        groups: newly_created

- name: Configure new instance
  hosts: newly_created
  become: yes
  roles:
    - common
    - nginx
    - myapp
```

---

**Q83. How do you implement blue-green deployment with Ansible?**
```yaml
- name: Blue-Green Deployment
  hosts: localhost

  vars:
    active_color: "{{ lookup('file', '/tmp/active_color') | default('blue') }}"
    new_color: "{{ 'green' if active_color == 'blue' else 'blue' }}"

  tasks:
    - name: Deploy to inactive color group
      include_playbook: deploy.yml
      vars:
        target_hosts: "{{ new_color }}_servers"
        version: "{{ deploy_version }}"

    - name: Smoke test new deployment
      uri:
        url: "http://{{ new_color }}-lb.internal/health"
        status_code: 200

    - name: Switch load balancer to new color
      command: lb-switch.sh {{ new_color }}

    - name: Record new active color
      copy:
        content: "{{ new_color }}"
        dest: /tmp/active_color
```

---

**Q84. How do you optimize Ansible for speed?**
```ini
# ansible.cfg performance tuning
[defaults]
forks = 50              # run on 50 hosts in parallel (default: 5)
gathering = smart       # only gather facts if not cached
fact_caching = redis    # cache facts in Redis
fact_caching_timeout = 3600  # cache for 1 hour

[ssh_connection]
pipelining = True       # reduces SSH connections (big speedup!)
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
control_path_dir = /tmp/.ansible/cp
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```
> **Biggest wins:** `pipelining = True` + increase `forks` + `gathering = smart`

---

**Q85. What is `ansible-pull`?**
```bash
ansible-pull -U https://github.com/org/ansible-playbooks.git playbook.yml
```
> Instead of control node pushing to managed nodes, each managed node PULLS and runs its own config. Used for:
> - Self-managing nodes
> - No central control node needed
> - Scaling to thousands of nodes (no SSH bottleneck)

---

**Q86. How do you handle Ansible in CI/CD?**
```yaml
# GitHub Actions
- name: Run Ansible playbook
  uses: dawidd6/action-ansible-playbook@v2
  with:
    playbook: playbook.yml
    directory: ansible/
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    inventory: |
      [webservers]
      ${{ secrets.PROD_SERVER_IP }}
    options: |
      --extra-vars "version=${{ github.ref_name }}"
      --vault-password-file .vault_pass
```

---

**Q87. What is `ansible-lint` and how to use it in CI?**
```yaml
# .github/workflows/ansible-lint.yml
- name: Lint Ansible
  uses: ansible/ansible-lint-action@main
  with:
    path: "playbooks/"
```
```bash
# Local use
pip install ansible-lint
ansible-lint playbook.yml
ansible-lint roles/
ansible-lint --profile production  # stricter rules
```

---

**Q88. How do you test Ansible playbooks with Molecule?**
```yaml
# molecule/default/molecule.yml
driver:
  name: docker
platforms:
  - name: ubuntu
    image: ubuntu:22.04
    pre_build_image: true
  - name: centos
    image: centos:8
    pre_build_image: true

provisioner:
  name: ansible
  playbooks:
    converge: converge.yml

verifier:
  name: ansible
```
```bash
molecule test     # full test cycle
# Runs: lint → create → prepare → converge → idempotency → verify → destroy
```

---

**Q89. What is the difference between Ansible and Terraform for infrastructure?**
> Use BOTH together:
> - **Terraform:** Create infrastructure (EC2, VPC, RDS, security groups)
> - **Ansible:** Configure what's inside (install software, deploy app, configure OS)
> ```
> terraform apply       → EC2 instances created
> ansible-playbook      → software installed and configured on EC2
> ```

---

**Q90. What are Ansible best practices?**
> 1. Use roles for reusable code
> 2. Use `defaults/main.yml` for role variables
> 3. Always use Ansible Vault for secrets
> 4. Use `--check` and `--diff` before applying in production
> 5. Use `serial` for rolling deployments
> 6. Enable `pipelining` for performance
> 7. Use `handlers` for service restarts
> 8. Tag tasks for selective running
> 9. Test with Molecule
> 10. Lint with ansible-lint in CI/CD
> 11. Use `no_log: true` for sensitive tasks
> 12. Keep inventory in version control (without secrets)

---

**Q91–Q110: Quick-fire important questions.**

**Q91. What is the `setup` module?**
> Collects facts about managed hosts. Runs automatically when `gather_facts: yes`. `ansible host -m setup` shows all facts.

**Q92. What does `state: present` vs `state: latest` mean in package modules?**
> - `state: present` — ensure package is installed (any version). Idempotent.
> - `state: latest` — ensure package is the latest version. May upgrade on every run.
> Use `present` in production (predictable), `latest` for dev.

**Q93. What is `ansible_connection: local`?**
> Run tasks on the control node itself instead of a remote host. Used for localhost automation or cloud API calls.

**Q94. What is `add_host` module?**
> Dynamically add a host to the inventory during playbook execution. Useful for provisioning new servers and immediately configuring them in the same playbook run.

**Q95. How do you run a playbook on localhost?**
```yaml
- hosts: localhost
  connection: local
  gather_facts: no
  tasks:
    - name: Local task
      command: echo "running locally"
```

**Q96. What is `with_nested` / nested loop?**
```yaml
- name: Give users access to databases
  command: grant-access.sh {{ item[0] }} {{ item[1] }}
  loop: "{{ ['alice', 'bob'] | product(['db1', 'db2']) | list }}"
```

**Q97. What is `group_by` module?**
> Dynamically create groups based on facts during playbook execution:
```yaml
- name: Group by OS
  group_by:
    key: "os_{{ ansible_distribution }}"
# Creates groups: os_Ubuntu, os_CentOS, etc.
```

**Q98. What is the `raw` module?**
> Runs a raw SSH command without Python dependency. Used to bootstrap Python on a fresh system:
```yaml
- name: Install Python (before any other modules work)
  raw: apt-get install -y python3
  become: yes
```

**Q99. What is `meta: flush_handlers`?**
```yaml
- meta: flush_handlers   # run pending handlers NOW instead of end of play
```
> Useful when you need handlers to run before a subsequent task (e.g., restart nginx, then health check).

**Q100. What is `check_mode`?**
```yaml
- name: This task only runs in check mode
  debug:
    msg: "Would have changed something"
  when: ansible_check_mode

- name: This task is skipped in check mode
  command: irreversible-operation.sh
  check_mode: no
```

**Q101. What is `listen` in handlers?**
```yaml
handlers:
  - name: Restart web services
    listen: "restart web"   # handlers can share a topic
    service:
      name: nginx
      state: restarted

  - name: Reload PHP
    listen: "restart web"
    service:
      name: php-fpm
      state: reloaded

tasks:
  - name: Update config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: "restart web"   # notifies ALL handlers listening to "restart web"
```

**Q102. What is the `slurp` module?**
> Read a file from remote host and return base64-encoded content:
```yaml
- slurp:
    src: /etc/myapp/version.txt
  register: version_file
- debug:
    msg: "Version: {{ version_file.content | b64decode }}"
```

**Q103. What is `ansible_managed`?**
> Special variable that inserts a warning in templated files:
```jinja2
# {{ ansible_managed }}
# DO NOT EDIT - This file is managed by Ansible
server_name = {{ server_name }}
```

**Q104. What is `environment` keyword in Ansible tasks?**
```yaml
- name: Run with custom environment
  command: npm install
  environment:
    NODE_ENV: production
    PATH: "/usr/local/bin:{{ ansible_env.PATH }}"
    HTTP_PROXY: "http://proxy.example.com:3128"
```

**Q105. What is `ANSIBLE_ROLES_PATH`?**
> Environment variable (or `roles_path` in ansible.cfg) that tells Ansible where to look for roles:
```ini
[defaults]
roles_path = ./roles:~/.ansible/roles:/etc/ansible/roles
```

**Q106. How do you handle multi-OS playbooks?**
```yaml
- name: Install package
  package:    # generic package module - works on any OS
    name: nginx
    state: present

# Or use include:
- include_tasks: "install_{{ ansible_os_family }}.yml"
# Loads install_Debian.yml or install_RedHat.yml
```

**Q107. What is the `expect` module?**
```yaml
- name: Answer interactive prompts
  expect:
    command: passwd alice
    responses:
      'New password:': 'secretpassword'
      'Retype new password:': 'secretpassword'
  no_log: true
```

**Q108. What is `throttle` in Ansible?**
```yaml
- name: Only 2 hosts run this at a time (even with high forks)
  command: db-migrate.sh
  throttle: 2
```

**Q109. What is `diff` mode?**
```bash
ansible-playbook playbook.yml --diff   # show file differences
```
> Shows what would change in files managed by `copy`, `template`, `lineinfile` etc. before actually changing them.

**Q110. Real-world scenario: Complete Ansible workflow for deploying a Node.js app.**
```yaml
---
- name: Deploy Node.js Application
  hosts: appservers
  become: yes
  serial: "50%"

  vars:
    app_name: myapp
    app_user: nodeapp
    app_dir: /opt/{{ app_name }}
    app_version: "{{ version | default('main') }}"
    node_port: 3000

  pre_tasks:
    - name: Remove from load balancer
      uri:
        url: "http://{{ lb_api }}/drain/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost
      ignore_errors: yes

  tasks:
    - name: Ensure app user exists
      user:
        name: "{{ app_user }}"
        shell: /bin/bash
        system: yes

    - name: Clone/update application
      git:
        repo: "https://{{ git_token }}@github.com/org/{{ app_name }}.git"
        dest: "{{ app_dir }}"
        version: "{{ app_version }}"
        force: yes
      become_user: "{{ app_user }}"

    - name: Install dependencies
      npm:
        path: "{{ app_dir }}"
        production: yes
      become_user: "{{ app_user }}"

    - name: Deploy environment config
      template:
        src: templates/env.j2
        dest: "{{ app_dir }}/.env"
        owner: "{{ app_user }}"
        mode: '0600'
      notify: Restart app

    - name: Deploy systemd service
      template:
        src: templates/app.service.j2
        dest: "/etc/systemd/system/{{ app_name }}.service"
      notify: Reload systemd

    - name: Start and enable service
      systemd:
        name: "{{ app_name }}"
        state: started
        enabled: yes
        daemon_reload: yes

    - name: Wait for app to be healthy
      uri:
        url: "http://localhost:{{ node_port }}/health"
        status_code: 200
      register: health
      until: health.status == 200
      retries: 12
      delay: 5

  post_tasks:
    - name: Add back to load balancer
      uri:
        url: "http://{{ lb_api }}/enable/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost

  handlers:
    - name: Reload systemd
      systemd:
        daemon_reload: yes

    - name: Restart app
      systemd:
        name: "{{ app_name }}"
        state: restarted
```

---

## 📅 Study Schedule

| Day | Topic | Status |
|-----|-------|--------|
| Day 1 | Jenkins | ✅ Done |
| Day 2 | AWS | ✅ Done |
| Day 3 | Docker | ✅ Done |
| Day 4 | Kubernetes | ✅ Done |
| Day 5 | Terraform | ✅ Done |
| Day 6 | Linux | ✅ Done |
| Day 7 | Git | ✅ Done |
| **Day 8** | **Ansible** | ✅ Done |
| Day 9 | Monitoring (Prometheus/Grafana) | ⏳ Next |
| Day 10 | Revision + Scenario Questions | ⏳ |

---

## 💡 Quick Tips for Ansible Interview

- **Most asked:** Playbook structure, roles, variables/vault, handlers, when conditions, idempotency
- **Always mention:** Vault for secrets, `--check` before production apply, `pipelining` for performance
- **Real experience:** "We used Ansible with rolling serial deployments to avoid downtime"
- **Know the difference:** `command` vs `shell`, `copy` vs `template`, `import` vs `include`
- **Trending:** Ansible with AWX/AAP, Molecule for testing, Ansible + Terraform together

---

*Almost there! 🎯 Day 8 complete. Just 2 more days! Next: Day 9 - Monitoring!*
