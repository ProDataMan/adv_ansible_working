## Set LAB_DIR environment variable
```
export LAB_DIR=$HOME/adv-ansible/labs/ansible-vault
```

## Use ansible-vault to encrypt the confidential file
Use `ansible-vault` to encrypt `$LAB_DIR/conf/confidential` to protect the confidential information stored within using the password "I love ansible".

* Run `ansible-vault encrypt $LAB_DIR/conf/confidential` and supply the password "I love ansible".

## Create a playbook that deploys apache2 on webservers 
Create playbook `$HOME/webserver.yml` that deploys `apache2` on webservers. It should be tagged with `base-install` and contain a handler that restarts the `apache2` daemon that is flagged by both installation and service manipulation for `apache2`.

* Create the file `$HOME/webserver.yml` and add the following content:
```yaml
 ---
 - hosts: webservers
   become: yes
   vars_files:
     - /home/ubuntu/adv-ansible/labs/ansible-vault/conf/confidential
   tasks:
     - name: install apache2
       apt:
         name: apache2
         state: latest
       notify: httpd service
       tags:
         - base-install
   handlers:
     - name: Restart and enable apache2
       service:
         name: apache2
         state: restarted
         enabled: yes
       listen: httpd service
```

## Deploy the templates
Configure `$HOME/webserver.yml` to deploy the templates `$LAB_DIR/conf/vhost.conf.j2` and `$LAB_DIR/conf/htpasswd.j2` to the `webservers` group. The `apache2` service must restart on config change. The tasks should be tagged `vhost`.

Add the following text to `$HOME/webserver.yml` just **before** the handler section:
```yaml
   - name: create apache config directory
       file:
         path: "/etc/httpd/conf.d"
         state: directory
         mode: '0755'
     - name: create another directory
       file:
         path: "/etc/httpd/conf"
         state: directory
         mode: '0755'
     - name: configure virtual host
       template:
         src: "/home/ubuntu/adv-ansible/labs/ansible-vault/conf/vhost.conf.j2"
         dest: "/etc/httpd/conf.d/vhost.conf"
       notify: httpd service
       tags:
         - vhost
     - name: configure site auth
       template:
         src: "/home/ubuntu/adv-ansible/labs/ansible-vault/conf/htpasswd.j2"
         dest: "/etc/httpd/conf/htpasswd"
       notify: httpd service
       tags:
         - vhost
```

## Asynchronously execute data-job on webservers
Configure `$HOME/webserver.yml` to asynchronously execute `/opt/data-job.sh` located on webservers with a timeout of `600` seconds and no polling. The task should be tagged with `data-job`.

Add the following text to `$HOME/webserver.yml` just before the handler section:
```yaml
- name: run data job
   command: /opt/data-job.sh
   async: 600
   poll: 0
   tags:
     - data-job
```

# Final `webserver.yml`
```yaml
---
 - hosts: webservers
   vars:
   become: yes
   vars_files:
     - "/home/ubuntu/adv-ansible/labs/ansible-vault/conf/confidential"
   tasks:
     - name: install httpd
       apt:
         name: apache2
         state: latest
       notify: httpd service
       tags:
         - base-install
     - name: create apache config directory
       file:
         path: "/etc/httpd/conf.d"
         state: directory
         mode: '0755'
     - name: create another directory
       file:
         path: "/etc/httpd/conf"
         state: directory
         mode: '0755'
     - name: configure virtual host
       template:
         src: "/home/ubuntu/adv-ansible/labs/ansible-vault/conf/vhost.conf.j2"
         dest: "/etc/httpd/conf.d/vhost.conf"
       notify: httpd service
       tags:
         - vhost
     - name: configure site auth
       template:
         src: "/home/ubuntu/adv-ansible/labs/ansible-vault/conf/htpasswd.j2"
         dest: "/etc/httpd/conf/htpasswd"
       notify: httpd service
       tags:
         - vhost
     - name: copy data job to all hosts
       copy:
         src: "/home/ubuntu/adv-ansible/labs/ansible-vault/bin/data-job.sh"
         dest: /opt/data-job.sh
         owner: ubuntu
         group: ubuntu
         mode: 755
     - name: run data job
       command: /opt/data-job.sh
       async: 600
       poll: 0
       tags:
         - data-job
   handlers:
     - name: Restart and enable httpd
       service:
         name: apache2
         state: restarted
         enabled: yes
       listen: httpd service
```

## Execute playbook to verify your playbook works correctly
Execute playbook `$HOME/webserver.yml` to verify your playbook works correctly.
Run the following command on the control node and provide the vault password "I love ansible".
```bash
ansible-playbook --ask-vault-pass $HOME/webserver.yml
```


