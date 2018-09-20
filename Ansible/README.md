### Ansible Knowledge
---
In the continuation of this workbook, we will install, 3 tiered application by ansible step by step and learn some of the important modules.
- nginx as loadbalancer
- apache2 as webserver for serving a python application
- mysql

### First of All, What is Ansible ?
---
- Ansible Configuration Management solution and not only a CM. 
- Simple automation language and engine that runs ansible playbooks YAML files.
- Agentless
- Writen in `Phyton`
- `Ansible Tower` in the enterprise framwork for controlling, securing and managing Ansible
- Ansibe has a logic what you would like to do is a change with the previous state and the current desired state. You will see it in playbooks and when you run multiple times it perform only the difference.
- Competitors: `Puppet`, `Chef`, `Saltstack`
- `Simple`, human readble, no special coding skills
- `Powerfull` and can orchestrate the complete app life cycle
- `Agentless`, uses OpenSSH and WinRM
---

### To Get Started And Learn More
---
- http://ansible.com/get-started
- http://ansible.com/community
- http://redhat.com/en/services/training/do407-automation-ansible
---

### The Ansible Way
---
- Cross Platform: Linux, Windows, Unix
- Human Readable: YAML and Jinja2 (Markup Language)
- Perfect Description Of Application: Ensuring everyone is on the same page.
- Version Controlled
- Dynamic Inventories: Capture all the servers % 100 of the time, regardless of infra, location etc.
- Orchastration That Play Weel With Others: HP SA, Pupet, Jenkins, RHNSS to homogenize existing environments by current toolsets and update mechanisms.
---

### Large community and wide variety of automation taks
---
- Over 1300 modules: cloud, containers, database, files, messaging, monitoring, network, notifications, packaging, source control, system, testing, utilizers, web infrastructure.
---

### It's not only a traditional CM tool:
It can be used and can perform;
---
- Application Deployment
- Multi-Tier Orchestration
- Provisioning
- Configuration Management
- Continuous Delivery
- Security & Compliance
- It's the only automation engine/language that can automate the entire application lifecycle and continuous delivery pipeline.
---

### Installation
---
- http://docs.ansible.com/ansible/installation

As ansible is a Python application, the best way is installing by pip. Byp pip you also get the newest versions instead of getting 8 versions back by package managers like yum and apt.
---
Installation
```
apt-get install python-pip
pip install ansible
```
Check Installation
```
ansible --version
ansible-playbook —version
ansible-galaxy —version
```
default config file= /etc/ansible/ansible.cfg
It can be over-ridden by creating a directory and put ansible.cfg and hosts (inventory) inside and keep at git for version conrol.

### How Ansible Works
---
- PLAYBOOKS: Written in `YAML` describing the changes.
- MODULES:
- PLUGINS: Mostly are modules that are running on control node for specific issues.
https://docs.ansible.com/ansible/2.5/plugins/plugins.html
- INVENTORY
---

### Modules
---
- http://docs.ansible.com
- apt/yum , copy , file , get_url , git, ping , debug , service , synchronize , template , uri , user, wait_for, assert
---
Get installed modules
```
ansible-doc -l
```
```
# OR Get ansible module location and list
ansible --version
ls 
1
/usr/lib/python2.7/dist-packages/ansible/modules/core/network/ios/ios_config.p

```

### Modules Run Commands
---
- `command`: run a OS command directly.
- `shell`: running a command by remote shell starting /bin/sh.
- `script`: run a script after transfering it.
- `raw`: Executes a command without going through the ansible module subsystem.
To work in ansiblist way, don't use non of them, as ansible works in a logic to keep the things in a desired state where described in playbooks. The seond time you run a playbook, if the description and the infrastructure matching, it will not do nothing. If you do a change and run again, only that change will. Modules are designed by this logic that's why should only be used.
---

### Ansible.cfg
You can set as default inventory file as dev.
```
$ cat /etc/ansible/ansible.cfg

[defaults]
inventory=./dev
```

### Inventory
---
Data that invetory holds;
- Hosts                               - Groups
- Inventory-specific data (variables) - Static or dynamic sources
---
```
$ cat ./dev

[control]
control ansible_host=localhost ansible_connection=localhost

[web]
node-[1:3] ansible_host=10.42.0.[6:8]

[haproxy]
haproxy ansible_host=10.42.0.100

[all:vars]
ansible_user=vagrant
ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```
```
ansible -i dev --list-hosts all
```

### Host Selection
You can use regex
```
ansible --list-hosts "*"
```
By group name
```
ansible --list-hosts loadbalancers
ansible --list-hosts "httpd*"
ansible --list-hosts loadbalancers:lnb1
```
Python Style
```
ansible --list-hosts webserver[0]
```
List all out of control group. All bash commands you need to use \ 
```
$ ansible --list-hosts \!control
```
- More Patterns
http://docs.ansible.com/ansible/info_patterns.html

### Tasks & Ad-Hoc Commands
It's not network ping, ansible ping, responds as pong.
It's a module in ansible there are modules.
command is a module for running command line.
It has large number of modules.
```
ansible -m ping all
ansible -m command -a "hostname" all
ansible -a "hostname" all
```
Error status, if the result is false, ansible returns red colored output else it's green.
```
$ ansible -a "/bin/false" all
```

### Ad-Hoc Commands
```
ansible all -m ping
ansible localhost -m setup  # gather_facts
ansible localhost -m command -a "uptime"
```

### Plays
Playbooks yaml syntax file. yaml is a simple markup language. 
```
$ mkdir playbooks
```
```
$ cat playbooks/hostname.yaml
---
  # Each Item in the list starts with - dash in yaml.
  - host: all
    tasks:
      - command: name=task
```

### Playbook Execution
```
$ cat playbooks/hostname.yml

---
  # Each Item in the list starts with - dash in yaml.
  - host: all
    tasks:
      - name: get server hostname
        command: hostname
```
```
ansibe=playbook playbooks/hostname.yml
```
```
Output details:
- PLAY [all]
- FACTS: Before it runs, it gatters some facts from each host.
- TASKS:
  changed: [httpd1]
  Changed means executed without errors.
  No awareness changed means executed.
- PLAY RECAP:
  Shows all the steps
```

### Playbook Introduction
You can do everything. Package, Source Repo, Service Hangle, Scripts, File, Directories, Users, Iptables. Make a list of things to configue to run your application. You do it step by step, run and add more steps, debug on the way. Built the last status.

Always check in which module you can do and how to use the module. And see the example below.
- https://docs.ansible.com/ansible/latest/modules/modules_by_category.html

### Module Packages: apt module, Install a cluster
```
$ cat loadbalancer.yml

- hosts: loadbalancers
  tasks: 
  - name: install nginx
    apt: name=nginx state=present update_cache=yes
    # removing
    apt: name=nginx state=absent update_cache=yes
```
```
$ cat database.yml
---
- hosts: databases
  tasks:
    - name: install mysql
      apt: name=mysql state=present update_cache=yes
```
### Packages: become
---
- Priviledge escalation module
- Become means runas root
- As installations should run as root. 
---
```
$ cat database.yml

---
- hosts: databases
  become: true
  tasks:
    - name: install mysql
      apt: name=mysql state=present update_cache=yes
```
```
ansible-playbook database.yml
```
The second run will fast and will not do nothing because we are in a desired state.
```
ansible-playbook database.yml
```
```
$ cat webserver.yml
---

### with_items and {{item}} 
also apt: module is used, in redhat yum:
---
- with_items and {{item}} injects the list of packages for multi package installation instead of creating task for each. with_items loop
- This usage is jinja syntax not yaml. 
- Jinja is templating language library.
- http://jinja.pocoo.org/
```
$ cat webserver.yml

---
- hosts: webservers
  become: true
  tasks:
    - name: install webserver
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - apache2
        - libapache2-mod-wsci
        - python-pip
        - python-virtualenv
        - python-mysqldb
```
```
ansible-playbook database.yml
```

### Service: service
```
$ cat loadbalancer.yml

---
- hosts: loadbalancers
  tasks: 
  - name: install nginx
    apt: name=nginx state=present update_cache=yes
    # removing
    apt: name=nginx state=absent update_cache=yes
  - name: ensure nginx started
    service: name=nginx state=started enabled=yes
    # Stop and disable. Also reload is possible.
    #service: name=nginx state=stoped enabled=no
```
- On control module where ansible running.
```
$ cat loadbalancer.yml

---
- hosts: control
  become: true
  tasks:
    - name: install tools
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - curl
```
```
ansible-playbook loadbalancer.yml
```

### Stack Restart

```
$ cat playbooks/stack_restart.yml

---
- hosts: loadbalancers
  become: true
  tasks:
    - service: name=nginx state=stopped

- hosts: webservers
  become: true
  tasks:
    - service: name=apache2 state=stopped

- hosts: databases
  become: true
  tasks:
    - service: name=mysql state=restarted

- hosts: loadbalancers
  become: true
  tasks:
    - service: name=nginx state=started

- hosts: webservers
  become: true
  tasks:
    - service: name=apache2 state=started
```
```
ansible-playbook playbooks/stack_restart.yml
```

### Notify and handlers
#### Services: apache2_module, handlers, notify
```
$ cat webserver.yml

---
- hosts: webservers
  become: true
  tasks:
    - name: install webserver
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - apache2
        - libapache2-mod-wsci
        - python-pip
        - python-virtualenv
        - python-mysqldb

    - name: ensure apache2 started
      service: name=apache2 state=started enabled=yes

    - name ensure mod_wsgi enabled
      apache2_module: state=present name=wsgi
      notify: restart apache2

  handlers:
  - name restart apache
    service: name=apache2 state=restarted
```
```
ansible-playbook webserver.yml
```
---
- If you use service restart, it will restart the service each time playbook is ran even you don't want to and there is no module change. 
- solution for this is `handlers and notify` if you don't want to restart. As you see above notify will run the handler if there is a change. 
---

### File Copy
```
    - name: copy demo app source
      copy: src=demo/app dest=/var/www/demo mode=0755
      notify: restart apache2
```
```
$ cat webserver.yml
- hosts: webservers
  become: true
  tasks:
    - name: install webserver
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - apache2
        - libapache2-mod-wsci
        - python-pip
        - python-virtualenv
        - python-mysqldb

    - name: ensure apache2 started
      service: name=apache2 state=started enabled=yes

    - name ensure mod_wsgi enabled
      apache2_module: state=present name=wsgi
      notify: restart apache2

    - name: copy demo app source
      copy: src=demo/app dest=/var/www/demo mode=0755
      notify: restart apache2

    - name: copy apache virtual host config
      copy: src=demo/demo.conf dest=/etc/apache2/site-available mode=0755    
      notify: restart apache2

  handlers:
  - name restart apache
    service: name=apache2 state=restarted
```
```
ansible-playbook webserver.yml
```

### pip module Application Module pip
- Pip is package management of python, it can create an isolated container, put all python dependencies on it.
- By default install before ansible 
```
$ cat control.yml

    - name: setup python virtualenv
      pip: requirements=/var/www/demo/requirements.txt virtualenv=/var/www/.venv
      notify: restart apache
```
```
$ cat webserver.yml

---
- hosts: webservers
  become: true
  tasks:
    - name: install webserver
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - apache2
        - libapache2-mod-wsci
        - python-pip
        - python-virtualenv
        - python-mysqldb

    - name: ensure apache2 started
      service: name=apache2 state=started enabled=yes

    - name ensure mod_wsgi enabled
      apache2_module: state=present name=wsgi
      notify: restart apache2

    - name: copy demo app source
      copy: src=demo/app dest=/var/www/demo mode=0755
      notify: restart apache2

    - name: copy apache virtual host config
      copy: src=demo/demo.conf dest=/etc/apache2/site-available mode=0755    
      notify: restart apache2

    - name: setup python virtualenv
      pip:       requirements=/var/www/demo/requirements.txt virtualenv=/var/www/.venv
      notify: restart apache

  handlers:
  - name restart apache
    service: name=apache2 state=restarted
```
```
ansible-playbook webserver.yml
```

### File, File
#### Disable apache default site and enable our new site.
```
    - name: de-activate default apache site
      file: path=/etc/apache2/sites-enabled/000-default.conf
      notify: restart apache

    - name: activate demo apache site
      file: src=/etc/apache2/sites-available/demo.conf dest=/etc/apache2/sites-enabled/demo.conf state=link
      notify: restart apache
```
```
$ cat webserver.yml

---
- hosts: webservers
  become: true
  tasks:
    - name: install webserver
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - apache2
        - libapache2-mod-wsci
        - python-pip
        - python-virtualenv
        - python-mysqldb

    - name: ensure apache2 started
      service: name=apache2 state=started enabled=yes

    - name ensure mod_wsgi enabled
      apache2_module: state=present name=wsgi
      notify: restart apache2

    - name: copy demo app source
      copy: src=demo/app dest=/var/www/demo mode=0755
      notify: restart apache2

    - name: copy apache virtual host config
      copy: src=demo/demo.conf dest=/etc/apache2/site-available mode=0755    
      notify: restart apache2

    - name: setup python virtualenv
      pip:       requirements=/var/www/demo/requirements.txt virtualenv=/var/www/.venv
      notify: restart apache

    - name: de-activate default apache site
      file: path=/etc/apache2/sites-enabled/000-default.conf
      notify: restart apache

    - name: activate demo apache site
      file: src=/etc/apache2/sites-available/demo.conf dest=/etc/apache2/sites-enabled/demo.conf state=link
      notify: restart apache

  handlers:
  - name restart apache
   state=restarted
```
```
ansible-playbook webserver.yml
```

### Files: template
$ mkdir templates

```
$ cat roles/nginx/templates/nginx.conf.j2

# Jinja format loop upstream
upstream demo {
{% for server in groups.webservers %}
  server {{ server }};
{% endfor %}

# Nginx conf format
server {
  listen 80;
  location / {
    proxy_pass http://demo;
  }
}
```
```
$ cat loadbalancer.yml

---
- hosts: loadbalancers
  become: true
  tasks:
  - name: install nginx
    apt: name=nginx state=present update_cache=yes

  - name: ensure nginx started
    service: name=nginx state=started enabled=yes

  - name: configure nginx site
    template: src=template/nginx.conf.j2 dest=/etc/nginx/sites-available/demo mode=0644
    notify: restart nginx

  - name: disable default nginx site
    file : src=/etc/nginx/sites-enabled/defaultdest=/etc/nginx/sites-available/default state=absent
    notify: restart nginx

  - name: enable nginx site
    file : src=/etc/nginx/sites-enabled/demodest=/etc/nginx/sites-available/demo state=link
    notify: restart nginx
  
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```
```
$ ansible-playbook loadbalancer.yml
```

### Files: lineinfile
### Change configuration inside a file.
```
   - name ensure mysql listening on all ports
     lineinfile: dest/etc/mysql/my.cnf regexp=^bind-address line="bind-address = 0.0.0.0"
     notify restart mysql

     handlers: 
     - name: restart mysql
       service name=mysql state=restarted  
```
```
$ cat database.yml

---
- hosts: databases
  become: true
  tasks:
    - name: install mysql
      apt: name=mysql state=present update_cache=yes

    - name: ensure mysql started
      service: name=mysql state=started enabled=yes

    - name ensure mysql listening on all ports
      lineinfile: dest/etc/mysql/my.cnf regexp=^bind-address line="bind-address = 0.0.0.0"
      notify restart mysql

  handlers: 
  - name: restart mysql
    service name=mysql status=restarted
```
```
ansible-playbook database.yml
```

### Application Modules: mysql_db, mysql_user
#### Configure mysql user and password.
```
- name create demo database
  mysql_db: name=demo state=present

- name create: mysql user
  mysql_user: name=demo pasword=demo priv=demo.*:ALL
```
```
$ cat database.yml

---
- hosts: databases
  become: true
  tasks:
    - name: install mysql
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - python-mysqldb

    - name: ensure mysql started
      service: name=mysql state=started enabled=yes

    - name ensure mysql listening on all ports
      lineinfile: dest/etc/mysql/my.cnf regexp=^bind-address line="bind-address = 0.0.0.0"
      notify restart mysql

    - name create demo database
      mysql_db: name=demo state=present

    - name create: mysql user
      mysql_user: name=demo pasword=demo priv=demo.*:ALL

  handlers: 
  - name: restart mysql
    service name=mysql status=restarted
```
```
ansible-playbook database.yml
```
### Support Playbook 2 - Stack Status: wait_for
- https://docs.ansible.com/ansible/wait_for_module.html

Create stack_status.yml playbook
```
$ cat playbooks/stack_status.yml

---
- hosts: loadbalancers
  become: true
  tasks:
    - name: verify nginx service
      command: service nginx status

- hosts: webservers
  become: true
  tasks:
    - name: verify apache2 service
      command: service apache2 status

    - name: verify apache2 listening on 80
      wait_for: port=80 timeout=1

- hosts: databases
  become: true
  tasks:
    - name: verify mysql is running
      command: service mysql status

    - name: check mysql listening on 3306
      wait_for: port: 3306 timeout=1

- hosts: control
  tasks:
    - name: verify end-to-end response
      uri: url=http://{{item}} return_content=yes
      with_items: groups.loadbalancers
      register: lb_index
```
```
ansible-playbook playbooks/stack_status.yml
```

Enchance stack_restart.yaml
Use wait_for till the conncetions to drain for the next step.
```
- hosts: loadbalancers
  become: true
  tasks:
    - service: name=nginx state=stopped
    - wait_for: port=80 state=drained
```
Use wait_for till port=80 not answering
```
- hosts: webservers
  become: true
  tasks:
    - service: name=apache2 state=stopped
    - wait_for: port=80 state=stopped
```

```
$ cat playbooks/stack_restart.yml

---
- hosts: loadbalancers
  become: true
  tasks:
    - service: name=nginx state=stopped
    - wait_for: port=80 state=drained

- hosts: webservers
  become: true
  tasks:
    - service: name=apache2 state=stopped
    - wait_for: port=80 state=stopped

- hosts: databases
  become: true
  tasks:
    - service: name=mysql state=restarted

- hosts: loadbalancers
  become: true
  tasks:
    - service: name=nginx state=started

- hosts: webservers
  become: true
  tasks:
    - service: name=apache2 state=started
```

### Support Playbook 2 - Stack Status: uri, register, fail, when, wait_for
---
- python-httplib2 is necessary on nginx and ansible - wait_for is a module to wait an amount of time for a port or file and timeout with timeout parameter. default is 300 seconds. control to run this. To check status of load balancer and behind load balancer.
- https://docs.ansible.com/ansible/uri_module.html
- https://docs.ansible.com/ansible/playbooks_conditionals.html#register-variables
- https://docs.ansible.com/ansible/playbooks_conditionals.html#the-when-statement
- https://docs.ansible.com/ansible/playbooks_loops.html#standard-loops
---
```
    - name: install tools
      apt: name={{intem}} state=present update_cache=yes
      with_items:
        - python-httplib2
```
Add python_httplib2 to control nodes to loadbalancers
```
$ cat loadbalancer.yml

- hosts: loadbalancers
  become: true
  tasks: 
  - name install tools
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - python-httplib2

  - name: install nginx
    apt: name=nginx state=present update_cache=yes

  - name: ensure nginx started
    service: name=nginx state=started enabled=yes

  - name: configure nginx site
    template: src=template/nginx.conf.j2 dest=/etc/nginx/sites-available/demo mode=0644
    notify: restart nginx

  - name: disable default nginx site
    file : src=/etc/nginx/sites-enabled/defaultdest=/etc/nginx/sites-available/default state=absent
    notify: restart nginx

  - name: enable nginx site
    file : src=/etc/nginx/sites-enabled/demodest=/etc/nginx/sites-available/demo state=link
    notify: restart nginx
  
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted

Add python_httplib2 to control nodes
```
```
$ cat control.yml

---
- hosts: control
  become: true
  tasks:
    - name: install tools
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - curl
        - python_httplib2
```
Add to playbooks/stack_status.yml
```
# for loadbalancers response
- hosts: control
  tasks:
    - name: verify end-to-end response
      uri: url=http://{{item}} return_content=yes
      with_items: groups.loadbalancers
      register: lb_index

    - fail: msg="index failed to return content"
      when: "'Hello, from sunny' not in item.content"
      with_items "{{lb_index.results}}"

# for webservers response
- hosts: loadbalancers
  tasks:
    - name: verify end-to-end response
      uri: url=http://{{item}} return_content=yes
      with_items: groups.webservers
      # content output
      register: app_index

      # fail
    - fail: msg="index failed to return content"
      # when to fail, if no error, it will skip the fail.
      when: "'Hello, from sunny' not in item.content"
      with_items "{{app_index.results}}"
```
```
$ cat playbooks/stack_status.yml

---
- hosts: loadbalancers
  become: true
  tasks:
    - name: verify nginx service
      command: service nginx status

- hosts: webservers
  become: true
  tasks:
    - name: verify apache2 service
      command: service apache2 status

    - name: verify apache2 listening on 80
      wait_for: port=80 timeout=1

- hosts: databases
  become: true
  tasks:
    - name: verify mysql is running
      command: service mysql status

    - name: check mysql listening on 3306
      wait_for: port: 3306 timeout=1

- hosts: control
  tasks:
    - name: verify end-to-end response
      uri: url=http://{{item}} return_content=yes
      with_items: groups.loadbalancers
      register: lb_index

# for loadbalancers response
- hosts: control
  tasks:
    - name: verify end-to-end response
      uri: url=http://{{item}} return_content=yes
      with_items: groups.loadbalancers
      register: lb_index

    - fail: msg="index failed to return content"
      when: "'Hello, from sunny' not in item.content"
      with_items "{{lb_index.results}}"

# for webservers response
- hosts: loadbalancers
  tasks:
    - name: verify end-to-end response
      uri: url=http://{{item}} return_content=yes
      with_items: groups.webservers
      # content output
      register: app_index

      # fail
    - fail: msg="index failed to return content"
      # when to fail, if no error, it will skip the fail.
      when: "'Hello, from sunny' not in item.content"
      with_items "{{app_index.results}}"

# for loadbalancers database response
- hosts: control
  tasks:
    - name: verify end-to-end response
      uri: url=http://{{item}} return_content=yes
      with_items: groups.loadbalancers
      register: lb_db_index

    - fail: msg="index failed to return content from db"
      when: "'A text that comes from database' not in item.content"
      with_items "{{lb_db_index.results}}"

# for webservers database response
- hosts: webservers
  tasks:
    - name: verify end-to-end response
      uri: url=http://{{item}} return_content=yes
      with_items: groups.loadbalancers
      # content output
      register: app_index

      # fail
    - fail: msg="index failed to return content from db"
      # when to fail, if no error, it will skip the fail.
      when: "'A text that comes from database' not in item.content"
      with_items "{{app_index.results}}"
```

### Roles
---
- Module Roles is like functions to define objects like files, handlers, tasks, template for using tasks.
- https://docs.ansible.com/ansible/playbooks_roles.html
- First create a folder called roles initialize a role for each tier.
---
```
ansible container init  # inits the main foldes to work with ansible.
ansible-galaxy init control # inits roles under the container.
ansible-galaxy init nginx
ansible-galaxy init apache2
ansible-galaxy init demo_app
ansible-galaxy init mysql
```

### Ansible galaxy
---
Above created roles with ansible-galaxy commands by our selves.
There is ansibe galaxy site which you can install pre-defined roles.
You can register, create and push your own roles galaxy site.
- https://galaxy.ansible.com/
How To:
- https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html
---
```
# Example Galaxy Commands
# You can use inside yaml as well.
$ ansible-galaxy install geerlingguy.apache,v1.0.0
$ ansible-galaxy install git+https://github.com/geerlingguy/ansible-role-apache.git,0b7cd353c0250e87a26e0499e59e7fd265cc2f25
$ ansible-galaxy install -r requirments.yml
```

### Converting to Roles: files, templates

```
# For control nodes
$ cat roles/control/tasks/main.yml

---
    - name: install tools
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - curl
        - python_httplib2
```
```
# For control nodes
$ cat control.yml

---
- hosts: control
  become: true
  roles:
    - control
```
```
# for databases
- hosts: databases
  become: true
  roles:
    - mysql
```
```
# for databases, tasks
$ cat roles/mysql/tasks/main.yml

---
    - name: install mysql
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - python-mysqldb

    - name: ensure mysql started
      service: name=mysql state=started enabled=yes

    - name ensure mysql listening on all ports
      lineinfile: dest/etc/mysql/my.cnf regexp=^bind-address line="bind-address = 0.0.0.0"
      notify restart mysql

    - name create demo database
      mysql_db: name=demo state=present

    - name create: mysql user
      mysql_user: name=demo pasword=demo priv=demo.*:ALL
```
```
# for databases, handlers
$ cat roles/mysql/handlers/main.yml 
---
  - name: restart mysql
    service name=mysql status=restarted
```

***
### Converting to roles, files, templates
#### NGINX
```
$ cp template/nginx.conf.j2 roles/nginx/templates/
```
```
$ cat roles/nginx/templates/nginx.conf.j2

# Jinja format loop upstream
upstream demo {
{% for server in groups.webservers %}
  server {{ server }};
{% endfor %}

# Nginx conf format
server {
  listen 80;
  location / {
    proxy_pass http://demo;
  }
}
```
```
$ cat loadbalancer.yml

---
- hosts: loadbalancers
  become: true
  tasks:
```

```
$ cat roles/nginx/tasks/main.yml

--- 
  - name install tools
      apt: name={{item}} state=present update_cache=yes
      with_items:
        - python-httplib2

  - name: install nginx
    apt: name=nginx state=present update_cache=yes

  - name: ensure nginx started
    service: name=nginx state=started enabled=yes

  - name: configure nginx site
    template: src=nginx.conf.j2 dest=/etc/nginx/sites-available/demo mode=0644
    notify: restart nginx

  - name: disable default nginx site
    file : src=/etc/nginx/sites-enabled/defaultdest=/etc/nginx/sites-available/default state=absent
    notify: restart nginx

  - name: enable nginx site
    file : src=/etc/nginx/sites-enabled/demodest=/etc/nginx/sites-available/demo state=link
    notify: restart nginx
```

```
$ cat roles/nginx/handlers/main.yml

---  
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```

#### For webservers, move some steps to demo_app role
#### Split into 2 different roles apache2 and demo_app for copy file etc.
```
$ cat webserver.yml

- hosts: webserver
  become: true
  roles:
    - apache2
    - demo_app
```

```
$ cat roles/apache2/tasks/main.yml

---
- name: install web components
  apt: name={{item}} state=present update_cache=yes
    with_items:
    - apache2
    - libapache2-mod-wsgi

- name: ensure mod_wsgi enabled
  apache2_module: state=present name=wsgi
  notify: restart apache2

- name: de-activate default apache site
  file: path=/etc/apache2/sites-enabled/000-default.conf state=absent
  notify: restart apache2

- name: ensure apache2 started
  service: name=apache2 state=started enabled=yes
```
```
$ cat roles/demo_app/tasks/main.yml

---
- name: install web components
  apt: name={{item}} state=present update_cache=yes
  with_items:
  - python-pip
  - python-virtualenv
  - python-mysqldb

- name: copy demo app source  
  copy: src=demo/app/ dest=/var/www/demo mode=0755
  notify: restart apache2

- name: copy apache virtual host config  
  copy: src=demo/demo.conf dest=/etc/apache2/sites-available mode=0755
  notify: restart apache2

- name: setup python virtualenv
  pip: requirements=/var/www/demo/requirements.txt virtualenv=/var/www/demo/.venv
        notify: restart apache2

- name: activate demo apache site
  file: src=/etc/apache2/sites-available/demo.conf dest=/etc/apache2/sites-enabled/demo.conf state=link
  notify: restart apache2
  notify: restart apache2
```

### NOTES: From this point no configuration will not be printed fully. All are just sample blocks.

```
$ cat roles/demo_app/handlers/main.yml

---
name: restart apache2
     service: name=apache2 state=restarted
```
```
$ cat roles/apache2/handlers/main.yml

---
name: restart apache2
     service: name=apache2 state=restarted
```

### Site.yml include
Include everything in one file. You can do incluse in tasks and handlers as well.
```
cat roles/site.yml

---
- include: control.yml
- include: database.yml
- include: webserver.yml
- include: loadbalancer.yml
```

### Variable: facts
Gathering facts and variables.
This will print all the information related to server in json format.
```
ansible -m setup db01
```
A section from roles/mysql/main.yml. Using variable from facts inside yaml to change bind ip in my.cnf conf to our ip. As an example mysql database ip address always changing.
```
# Add to database.yml
- hosts: webservers
  gather_facts: false
```
```
# Add to playbooks/stack_status.yml
- hosts: webservers
  gather_facts: false
```
```
$ cat roles/mysql/site.yml

- name: ensure mysql listening on all ports
  lineinfile: dest=/etc/mysql/my.cnf regexp=^bind-address
              line="bind-address = {{ ansible_eth0.ip4.address }}"
```
```
$ cat playbooks/stack_restart.yml

- name: verify mysql is listening on 3306
  wait_for: host={{ ansible_eth0.ipv4.address }} port=3306 state=started
```

### Variables: defaults
```
$ cat roles/mysql/main.yml

- name: create database
  mysql_db: name={{ db_name }} state=present

- name: create user
  mysql_user: name={{ db_user_name }} password={{ db_user_pass }} priv={{ db_name }}.*:ALL host='%'
              host='{{ db_user_host }}' state=present
```
Variables are set in defaults folder under roles.
```
$ cat roles/mysql/defaults/main.yml

db_name: mypp
db_user_name: dbuser
db_user_pass: dbpass
db_user_host: local
```

### Variables vars
---
The previous were defaults specific to app. Use defaults.
And it's the good practice using vars and keeping.
- https://docs.ansible.com/ansible/2.6/user_guide/playbooks_variables.html
---
```
$ cat roles/mysql/vars/main.yml
```
---
You can set vars everywhere and ansible has an hierarchical over-riding variables. Explained here.
- https://docs.ansible.com/ansible/2.6/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable
---
```
# From Least To Gratest. extra vars wins because cannot be overridden.
role defaults [1]
inventory file or script group vars [2]
inventory group_vars/all [3]
playbook group_vars/all [3]
inventory group_vars/* [3]
playbook group_vars/* [3]
inventory file or script host vars [2]
inventory host_vars/* [3]
playbook host_vars/* [3]
host facts / cached set_facts [4]
play vars
play vars_prompt
play vars_files
role vars (defined in role/vars/main.yml)
block vars (only for tasks in block)
task vars (only for the task)
include_vars
set_facts / registered vars
role (and include_role) params
include params
extra vars (always win precedence)
```

Overriding/injecting variables when calling a role
```
$ cat database.yml 

---
- hosts: database
  become: true
  roles:
    - role: mysql
      db_user_name: "{{ db_user }}"
      db_user_pass: "{{ db_pass }}"
      db_user_host: '%'
```

### Variables: with_dict
- Example with_dict usage:
- sites: has a dictionary inside
- For each key in dictionary run jinja2 manipulate template and add to sites-available and separately adds soft line to sites-available
- its like with_items but dictionary is used instead of item.
- So, there will be 2 loops one for groups.webserver.
- The second key value loop. So we configure backend and frontend sites in enginx
Example: For each key in dictionary

```
$ cat roles/nginx/defaults/main.yml

sites:
  myapp:
    frontend: 80
    backend: 80
```
```
- name: configure nginx sites
  template: src=nginx.conf.j2 dest=/etc/nginx/sites-available/{{ item.key }} mode=0644
  with_dict: sites
  notify: restart nginx

- name: activate nginx sites
  file: src=/etc/nginx/sites-available/{{ item.key }} dest=/etc/nginx/sites-enabled/{{ item.key }} state=link
  with_dict: sites
  notify: restart nginx
```
```
$ cat roles/nginx/templates/nginx.conf.j2

upstream {{ item.key }} {
{% for server in groups.webserver %}
    server {{ server }}:{{ item.value.backend }};
{% endfor %}
}

server {
    listen {{ item.value.frontend }};

    location / {
        proxy_pass http://{{ item.key }};
    }
}
```

### Selective Removal: shell, register, with_items, Example:
---
- ls files inenabled-sites dir and save as variable active
- run a loop for each value in active variable
- when there is not a key in sites list, delete the file from configuration directory available sites
---
```
$ cat roles/nginx/defaults/main.yml

---
sites:
  myapp:
    frontend: 80
    backend: 80
```
```
$ cat roles/nginx/tasks/main.yml

- name: get active sites
  shell: ls -1 /etc/nginx/sites-enabled
  register: active

- name: de-activate sites
  file: path=/etc/nginx/sites-enabled/{{ item }} state=absent
  with_items: active.stdout_lines
  when: item not in sites
  notify: restart nginx
```

### Variables - continued
---
Example: We have a python db connection jinja2 template
- What we do is. We have variables. Calling the role by variables, so it's injecting  variables to the template while creating it.
---
```
activate_this = '/var/www/demo/.venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

import os
os.environ['DATABASE_URI'] = 'mysql://{{ db_user }}:{{ db_pass }}@{{ groups.database[0] }}/{{ db_name }}'

import sys
sys.path.insert(0, '/var/www/demo')

from demo import app as application
```
```
$ cat webserver.yml

---
- hosts: webserver
  become: true
  roles:
    - apache2
    - { role: demo_app, db_user: demo, db_pass: demo, db_name: demo }
```

### Variables; vars_files, group_vars
---
Those are the locations where all playbooks and playbooks uses without adding to ansible.cfg. Default locations. Good way to hold varibles in a central location.
- https://docs.ansible.com/ansible/intro_inventory.html#splitting-out-host-and-group-specific-data
- https://docs.ansible.com/ansible/playbooks_variables.html#variable-file-separation
---

Create directories
```
mkdir vars_files
mkdir group_var
touch group_var/all
```
Set variables
```
$ cat group_vars/all

---
db_name: demo
db_user: demo
db_pass: demo
```
Inject variables to roles
```
$ cat databases.yml
---
- hosts: database
  become: true
  roles:
    - role: mysql
      db_user_name: "{{ db_user }}"
      db_user_pass: "{{ db_pass }}"
      db_user_host: '%'
```
Use variables inside tasks code. This is called variable routing.
```
$ cat roles/mysql/tasks/main.yml

- name: create database
  mysql_db: name={{ db_name }} state=present

- name: create user
  mysql_user: name={{ db_user_name }} password={{ db_user_pass }} priv={{ db_name }}.*:ALL
              host='{{ db_user_host }}' state=present
```

### Variables: vault
---
Encrypting clear text for passwords, keys and important data to eliminate passwords inside yaml code.
- https://docs.ansible.com/ansible/2.6/user_guide/vault.html
---
```
# In the main directory that we are working
cd group_vars
mv all vars
mkdir all
mv vars all
export EDITOR=vi
cd all 
```
Continuation
```
$ ansible-vault create vault  
# vault in the end is the file name we create, it will ask password.
# vi editor will open
---
vault_db_pass: MyPassword123
``` 
To edit password. You need to remember the password which you gave to the file.
```
ansible-vault edit vault
```
Run playbooks giving password, else it will not work.
```
ansible-playbook --ask-vault-pass
```
Second way. By choosing vault file.
```
ansible-playbook --vault-password-file
```
The third way. This way will not ask password.
```
echo "mypassword123" > ~/.vault_pass.txt
chmod 0600 ~/.vault_pass.txt
```
Also you need to add this to ansible.cfg
```
$cat ansible.cfg

[defaults]
inventory = ./hosts
vault_password_file = ~/.vault_pass.txt
```

### External Roles & Galaxy
Galaxy: Online repository for roles.
https://galaxy.ansible.com
Check score and rating when using. Also time modified and created.
```
ansible-galaxy install username.rolename
```

### Advanced Execution Introduction
This topic covers execution time and reduring run time.
Also it covers writing less code.
- time command is user to measure
```
time ansible-playbook site.yml
```

#### Removing Unnecessary Steps:
---
- gather_facts: false
- In database mysql we don't do false because we are using variables .
---
```
- hosts: loadbalancer
  gather_facts: false
```

#### Extracting 
---
- cache_valid_time 
- example in this example we run update_cache with cache_valid_time on all hosts. Doing the caching one time. So we can remove update_cache=yes parameter in our entire code, instead of running it multiple times in all apt: tasks.
---
```
$ cat site.yml

---
- hosts all
  become: true
  gather_facts: true
  tasks:
    - name: update apt cache
      apt: update_cache=yes cache_valid_time=86400

- include: control.yml
- include: database.yml
- include: webserver.yml
- include: loadbalancer.yml
```

#### Limiting Execution by Hosts: limit
---
- If you want to do a change in one host inside a site.yml
- Or did a change only in one tear and you don't want the entires site.yml hosts action will be taken. Only the relevant actions.
---
```
ansible-playbook site.yml --limit app01   # regex will work as well
```

#### Limiting Execution by tags: 
---
- Tagging a tasks
- Additionally you can tag multiple tasks with same tag like system, configure, packages. So when you run ansible-playbook only those tagged tasks will run.
---
```
$ cat roles/control/tasks/main.yml

---
- name: install tools
  apt: name={{ item }} state=present
  with_items:
    - curl
    - python-httplib2
  tags [ 'packages' ]
```
```
ansible-playbook site.yml --list-tags
```
Run only defined tag.
```
ansible-playbook site.yml --tags "packages"
#
ansible-playbook site.yml --skip-tags "packages"
```

#### Idempotence: when_changed, when_failed
---
- changed_when: is used to manipulating changed status with a condition.
- failed_when: is used to manipulating failed status with a condition.
- They work use the same login.
- It effects how the playbook run works.
---
```
# Not from our codes.

tasks:
  - shell: /usr/bin/billybass --mode="take me to the river"
    register: bass_result
    changed_when: "bass_result.rc != 2
```
```
- name: get active sites
  shell: ls -1 /etc/nginx/sites-enabled
  register: active
  changed_when: "active.stdout_lines != sites.keys()"
```
```
# Not from our codes.
# This will never repost changed status

tasks:
  - shell: /usr/bin/billybass --mode="take me to the river"
    register: bass_result
    changed_when: false
```

### Accelerated Mode and Pipelining
#### Accelerated Mode
---
- Ansible is a python app which uses ssh. 
- https://docs.ansible.com/ansible/2.3/playbooks_acceleration.html
- Not reccommended what it does is keepalive ssh connection.
---
```
- hosts:
  accelerate: true
```

#### Pipelining
Also this you don't need to use if you don't have a very large code based environment.
```
$cat ansible.cfg

[defaults]
inventory = ./hosts
vault_password_file = ~/.vault_pass.txt

[ssh_connection]
pipelining = True
```
```
# This make fasters the sudo become opreations
echo "Defaults	!requiretty" >> /etc/sudoers
# OR
visudo
```
```
echo "Defaults	!requiretty" >> /etc/sudoers
```

### Module block:
- https://docs.ansible.com/ansible/2.6/user_guide/playbooks_blocks.html
```
tasks:
 - name: Attempt and graceful roll back demo
   block:
     - debug:
         msg: 'I execute normally'
     - command: /bin/false
     - debug:
         msg: 'I never execute, due to the above task failing'
   rescue:
     - debug:
         msg: 'I caught an error'
     - command: /bin/false
     - debug:
         msg: 'I also never execute :-('
   always:
     - debug:
         msg: "This always executes"
```

### Troubleshooting Ordering Problems
---
- As and example you have 2 tasks, first is restart the service and second configure the service.
- If you do mistake in configuration section, next playbook run new configuration will not load.
- The next run it will fail to restart, as it fail.
- You will fix the configuration change, it will not work because service restart is before, due to old configuration, service restart will throw error and configuration change will not work.
---

#### How to fix this ?
There are many ways. It depends what to use according to the situation.
---
1) Moving the restart task after configuration, which is more logical
2) commenting in for only one run the erroring section.
3) Adding ignore_errors: true to the service restart task
ignore_errors: true
---

### Jumping to Specific Tasks: list-tasks, step, start-at-task
Cause ansible-playbook command to get interactive y or no approval for each step
```
ansible-playbook site.yml --step
```
List tasks of a playbook
```
ansible-playbook site.yml --list-tasks
```
Start a specific task
```
ansible-playbook site.yml --start-at-task "name of the task"
```

### Retrying Failed Hosts
If a task fails like inside site.yml. It will create a file for failed tasks like @/home/ansible/site.retry and print in the output. To retry the failing tasks, run.
```
ansible-playbook site.yml --limit @/home/ansible/site.retry
```

### Syntax-Check & Dry-Run: syntax-check, check
Before running your code, do syntax check.
```
ansible-playbook --syntax-check site.yml
```
Simulate the run by --check before running.
```
ansible-playbook --check site.yml
```

### Debugging: debug
As an example below, debug will print out debug info in the end.
```
- name: get active sites
  shell: ls -1 /etc/nginx/sites-enabled
  register: active
  changed_when: "active.stdout_lines != sites.keys()"

- debug: var=active.stdout_lines
```

### YAML
---
YAML is a human frindly data serialization standard for all programming languages. Easier than JSON.
- http://yaml.org/
- yaml.org/start.html
- http://learnxinyminutes.com/docs/yaml/
- indentations are spaces, data includes key/value pairs and lists.
- Lists starts with 
---
```
#### Lists: May be in 2 formats.
product: [ item1, item2 ]
#OR
product:
  - item1
  - item2
```
#### Block Format 
Check | in the code to understand block format.
```
address:
  lines: |
      458 Wlakman St.
      Suite #292
```
 
#### Anchors
```
# See how &id001 and *id001 is used
bill-to : &id002
  given: Chris
  family Dumars
ship-to: *id001
- sku: BL394D
  quantity : 4
```

#### Jinja2 Templating Engine
---
- http://jinja.pocoo.org/
- You don't need to be an expert but learn the basics.
- Whenever you inject a variable, Jinja2 does the subout before.
---
```
{% extends "layout.html" %}
{% block body %}
  <ul>
  {% for user in users %}
    <li><a href="{{ user.url }}">{{ user.username }}</a></li>
  {% endfor %}
  </ul>
{% endblock %}
```

### Jinja2 From Ansible's Perspective
#### Template Designer
- http://jinja.pocoo.org/docs/2.10/templates/
```
<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Webpage</title>
</head>
<body>
    <ul id="navigation">
    {% for item in navigation %}
        <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
    {% endfor %}
    </ul>

    <h1>My Webpage</h1>
    {{ a_variable }}

    {# a comment #}
</body>
</html>
```

#### Filters
- http://jinja.pocoo.org/docs/2.10/templates/#filters

#### Tests
- http://jinja.pocoo.org/docs/2.10/templates/#tests

#### List of control structures
- Learn If, Loop structures
http://jinja.pocoo.org/docs/2.10/templates/#list-of-control-structures

