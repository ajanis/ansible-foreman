Ansible-Foreman
=========

Set up a Foreman Server as part of the Foreman build server Project

This role is part of a project that will configure a Foreman build environment with optional TFTP and DHCP smart proxies, and an NGINX webserver for serving static content and acting as a reverse-proxy to Foreman.

Requirements
------------

This role uses the [Foreman-Ansible-Modules](https://github.com/theforeman/foreman-ansible-modules) to configure the Foreman server.

Role Variables
--------------
```
foreman_start_service: True
foreman_proxy_port: 8000
foreman_config_dir: /etc/foreman
foreman_admin_password: "{{ vault_foreman_admin_password }}"
www_domain: "{{ ansible_domain }}"
foreman_hostname: "{{ ansible_hostname }}"
foreman_interface: 0.0.0.0
foreman_port: 3000
foreman_user: foreman
foreman_env: production

foreman_proxy_config_dir: /etc/foreman-proxy
foreman_proxy_user: foreman-proxy
foreman_proxy_group: foreman-proxy
foreman_proxy_port: 8000
foreman_proxy_protocol: http
foreman_proxy_bind_host: "{{ ansible_default_ipv4.address }}"

foreman_proxy_foreman_url: "http://127.0.0.1"

foreman_proxy_dhcp: true
foreman_proxy_dhcp_protocol: http
foreman_proxy_dhcp_server: 127.0.0.1
foreman_proxy_dhcp_subnets: "[]"
foreman_proxy_dhcp_omapi_port: 7911

foreman_proxy_tftp: true
foreman_proxy_tftp_protocol: http
foreman_proxy_tftp_dir: /srv/tftp
foreman_proxy_tftp_pxe_dir:
  - boot
  - pxelinux.cfg
```
Dependencies
------------

Additonal variables may be set in group_vars for configuring nginx, isc-dhcp server, and tftp server

Example Group Variables
----------
```
www_domain: home.example.com
foreman_hostname: foreman

fail2ban_enable: False

nginx_backends:
  - service: foreman
    servers:
      - localhost:3000

nginx_vhosts:
  - servername: "{{ foreman_hostname | default(ansible_hostname) + '.' + www_domain | default(ansible_domain) }}"
    serveralias: "{{ ansible_default_ipv4.address }} {{ ansible_fqdn }}"
    serverlisten: 80
    locations:
      - name: /
        proxy: True
        backend: foreman
      - name: 404.html
        docroot: "/usr/share/nginx/html"
      - name: 50x.html
        docroot: "/usr/share/nginx/html"

nginx_vhosts_ssl: []

isc_dhcp_server_subnet:
  - netaddress: 192.168.121.0
    netmask: 255.255.255.0
    gateway: 192.168.121.1
    domain: "{{ www_domain | default(ansible_domain) }}"
    domain_search: "{{ www_domain | default(ansible_domain) }}"
    dns: 192.168.121.1
    range: 192.168.121.20 192.168.121.100
```

Example Playbook
----------------
```
- name: "Deploy Foreman Server"
  hosts: foreman
  become: True
  remote_user: root
  tasks:


    - include_role:
        name: foreman
      tags:
        - install
        - configure
        - foreman
  
    - include_role:
        name: nginx
  
    - include_role:
        name: isc_dhcp_server
      when: foreman_proxy_dhcp

    - include_role:
        name: tftp
      when: foreman_proxy_tftp
```

License
-------

MIT

Author Information
------------------

Created by Alan Janis
