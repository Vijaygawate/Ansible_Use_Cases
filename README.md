# Ansible_Use_Cases

**Ansible for server provisioning **:

It will make your server ready with all the pakages , instead of running/installing commands manually one by one, we can specify commands and pacakges which are needed to provison server in ansible playbook.

'provisioning' just means setting up a server ready to accommodate whatever websites/web apps you wish to put on it.

Refer below playbook for server provisoning 
===============================================================================================
---
- name: Ubuntu 20.04 LAMP stack
  user: root
  hosts: all
  become: yes
  tasks:
    - name: Update repository list and cache
      apt: update_cache=yes cache_valid_time=3600
      
    - name: Upgrade all packages on the Cloud Server
      apt: upgrade=yes
      
    - name: Install the latest versions of each component
      apt:
        name:
          - apache2
          - mariadb-server
          - php
          - php-mysql
        state: latest

    - name: Check that apache2 is enabled and running
      service:
        name: apache2
        enabled: true
        state: started

    - name: Check that mariadb is enabled and running
      service:
        name: mariadb
        enabled: true
        state: started

    - name: Copy the php test page from remote
      get_url:
        url: "https://www.middlewareinventory.com/index.php"
        dest: /var/www/html/index.php
        mode: 0644

    - name: Confirm that the web server is working
      uri:
        url: http://{{ansible_hostname}}/index.php
        status_code: 200
===============================================================================================


**How to use Ansible to patch systems and install applications**

the yum module to update the system. Ansible can install, update, remove, or install from another location
==========================================
 - name: update the system
    yum:
      name: "*"
      state: latest
==========================================
After updating the system, we need to restart and reconnect:
 - name: restart system to reboot to newest kernel
    shell: "sleep 5 && reboot"
    async: 1
    poll: 0

  - name: wait for 10 seconds
    pause:
      seconds: 10

  - name: wait for the system to reboot
    wait_for_connection:
      connect_timeout: 20
      sleep: 5
      delay: 5
      timeout: 60

  - name: install epel-release
    yum:
      name: epel-release
      state: latest
==========================================
So far we've learned how to update a system, restart the VM, reconnect, and install a RPM. Next we will install NGINX using the role in Ansible Lightbulb.

 - name: Ensure nginx packages are present
    yum:
      name: nginx, python-pip, python-devel, devel
      state: present
    notify: restart-nginx-service

  - name: Ensure uwsgi package is present
    pip:
      name: uwsgi
      state: present
    notify: restart-nginx-service

  - name: Ensure latest default.conf is present
    template:
      src: templates/nginx.conf.j2
      dest: /etc/nginx/nginx.conf
      backup: yes
    notify: restart-nginx-service

  - name: Ensure latest index.html is present
    template:
      src: templates/index.html.j2
      dest: /usr/share/nginx/html/index.html

  - name: Ensure nginx service is started and enabled
    service:
      name: nginx
      state: started
      enabled: yes

  - name: Ensure proper response from localhost can be received
    uri:
      url: "http://localhost:80/"
      return_content: yes
    register: response
    until: 'nginx_test_message in response.content'
    retries: 10
    delay: 1
# handlers file for nginx-example
  - name: restart-nginx-service
    service:
      name: nginx
=====================================================================================================================================================
