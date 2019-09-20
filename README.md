# ec_apache_ansible
## Ansible - This is an ansible playbook written to launch an ec2 with httpd installed on it.

```
---
 - name: "EC2_LAMP"
   hosts: localhost
   gather_facts: False
   tasks:
     - name: "Launch new EC2"
       local_action: ec2
                     group=default
                     instance_type=t2.micro
                     image=ami-0d8f6eb4f641ef691
                     wait=true
                     region=us-east-2
                     keypair=ansible
                     count=1
       register: ec2

     - debug:
         var: item.public_ip
       with_items: "{{ ec2.instances }}"

     - name: "Adding host"
       add_host:
         hostname: webserver
         ansible_host: "{{ ec2.instances.0.public_ip }}"
         ansible_port: 22
         ansible_user: "ec2-user"
         ansible_ssh_private_key_file: "/home/ec2-user/ansible.pem"
         ansible_ssh_common_args: "-o StrictHostKeyChecking=no"

     - name: "wait for ec2 instance to come online"
       wait_for:
         port: 22
         host: "{{ ec2.instances.0.public_ip }}"
         timeout: 80
         state: started
         delay: 10
         
     - name: "Setup Apache webserver"
       delegate_to: webserver
       become: yes
       yum:
         name:
            - php-mysql
            - httpd
            - mariadb-server
            - php
            - MySQL-python

     - name: "Apache - Creating index.html"
       delegate_to: webserver
       become: yes
       copy:
         content: " <h1>It works..!!</h1>"
         dest: /var/www/html/index.html

     - name: "Apache - Restarting/Enabling"
       delegate_to: webserver
       become: yes
       service:
         name: "{{ item }}"
         state: restarted
       with_items:
         - httpd
         - mariadb
```
