# Ansible-Foreman

<!-- MarkdownTOC -->

- Requirements
- Role Variables
  - defaults/main.yml
  - vars/debian.yml
  - vars/redhat.yml
- Dependencies
- Example Group Variables for Supporting Roles
- Example Playbook
- License
- Author Information

<!-- /MarkdownTOC -->

Set up a Foreman Server as part of the Foreman build server Project

This role is part of a project that will configure a Foreman build environment with optional TFTP and DHCP smart proxies, and an NGINX webserver for serving static content and acting as a reverse-proxy to Foreman.

Database backends can also be deployed in docker containers for ease of maintenance.

## Requirements

Foreman configuration is handled by [Foreman-Ansible-Modules](https://github.com/theforeman/foreman-ansible-modules) to configure the Foreman server.

The initial configuration of the foreman smart-proxy is handled using the foreman-yml project

Several supporting services (such as Nginx, MySQL, PostgreSQL) can be deployed in containers which would require the [ansible-docker](https://github.com/ajanis/ansible-docker.git) role

## Role Variables

### defaults/main.yml
```
www_domain: "{{ ansible_domain }}"

foreman_enable_service: True
foreman_hostname: "{{ ansible_hostname }}"
foreman_interface: 0.0.0.0
foreman_port: 3000
foreman_config_dir: /etc/foreman
foreman_admin_user: foreman
foreman_admin_password: "{{ vault_foreman_admin_password | default('password') }}"
foreman_env: production

foreman_proxy_enable_service: True
foreman_proxy_config_dir: /etc/foreman-proxy
foreman_proxy_user: foreman-proxy
foreman_proxy_group: foreman-proxy
foreman_proxy_protocol: http
foreman_proxy_bind_host: "{{ ansible_default_ipv4.address }}"
foreman_proxy_port: 8000

foreman_proxy_dhcp: True
foreman_proxy_dhcp_protocol: http
foreman_proxy_dhcp_server: 127.0.0.1
foreman_proxy_dhcp_omapi_port: 7911
foreman_proxy_dhcp_subnets: []

foreman_proxy_tftp: True
foreman_proxy_tftp_protocol: http
foreman_proxy_tftp_dir: /srv/tftp
foreman_proxy_tftp_pxe_dir:
  - boot
  - pxelinux.cfg

foreman_db_adapter: postgresql
foreman_db_host: localhost
foreman_db_name: foreman
foreman_db_username: foreman
foreman_db_password: "{{ vault_foreman_db_password | default('password') }}"
foreman_db_timeout: 5000
foreman_db_pool: 5
foreman_db_sqlite_dir: /var/lib/foreman/db

foreman_yml_template: foreman_config.yml.j2
foreman_perform_rake_tasks: False
```

### vars/debian.yml
```
foreman_gpg_key_url: https://deb.theforeman.org/pubkey.gpg
foreman_repo_url: http://deb.theforeman.org/ 
foreman_install_version: 1.22

python_pip_pkg: python-pip
foreman_python_pkgs:
  - apypie
  - foreman-yaml
  - requests
  - foreman-yml
  - foreman
  - PyYAML

foreman_dependency_pkgs:
  - ca-certificates
  - python-httplib2
  - python-apt
  - python-setuptools
  - curl
  - apt-transport-https
  - wget

foreman_pkgs:
  - foreman
  - foreman-sqlite3
  - foreman-libvirt
  - foreman-console
  - foreman-debug
  - foreman-journald
  - foreman-cli
  - ruby-foreman
  - ruby-hammer-cli
  - ruby-hammer-cli-foreman
  - ruby-foreman-ansible-core
  - ruby-foreman-ansible
  - ruby-hammer-cli-foreman-ansible
  - ruby-foreman-templates
  - ruby-foreman-tasks
  - ruby-foreman-remote-execution
  - ruby-foreman-remote-execution-core
  - ruby-foreman-setup
  - ruby-foreman-discovery
  - ruby-foreman-bootdisk
  - ruby-foreman-dhcp-browser

foreman_proxy_pkgs:
  - foreman-proxy
  - foreman-proxy-journald


foreman_proxy_extra_repos_pkg: []

foreman_proxy_dhcp_config_dir: /etc/dhcp
foreman_proxy_dhcp_config: /etc/dhcp/dhcpd.conf
foreman_proxy_dhcp_leases: /var/lib/dhcp/dhcpd.leases

foreman_db_sqlite_adapter_pkg: foreman-sqlite3
foreman_db_mysql_adapter_pkg: foreman-mysql2
foreman_db_pgsql_adapter_pkg: foreman-postgresql

foreman_db_socket: /var/run/mysqld/mysqld.sock
```

### vars/redhat.yml
```
foreman_install_version: 1.22

foreman_db_sqlite_adapter_pkg: foreman-sqlite
foreman_db_mysql_adapter_pkg: foreman-mysql2
foreman_db_pgsql_adapter_pkg: foreman-postgresql
foreman_db_socket: /var/lib/mysql/mysql.sock 

python_pip_pkg: python-pip

foreman_dependency_pkgs:
  - ca-certificates
  - python-httplib2
  - python-setuptools
  - curl
  - wget
  - gcc
  - python-devel
  - python-setuptools

foreman_python_pkgs:
  - apypie
  - foreman-yaml
  - requests
  - foreman-yml
  - foreman
  - PyYAML

foreman_pkgs:
  - foreman
  - foreman-sqlite3
  - foreman-libvirt
  - foreman-console
  - foreman-debug
  - foreman-journald
  - foreman-cli
  - ruby-foreman
  - ruby-hammer-cli
  - ruby-hammer-cli-foreman
  - ruby-foreman-ansible-core
  - ruby-foreman-ansible
  - ruby-hammer-cli-foreman-ansible
  - ruby-foreman-templates
  - ruby-foreman-tasks
  - ruby-foreman-remote-execution
  - ruby-foreman-remote-execution-core
  - ruby-foreman-setup
  - ruby-foreman-discovery
  - ruby-foreman-bootdisk
  - ruby-foreman-dhcp-browser

foreman_proxy_pkgs:
  - foreman-proxy
  - foreman-proxy-journald


foreman_proxy_extra_repos_pkg: []

foreman_proxy_dhcp_config_dir: /etc/dhcp
foreman_proxy_dhcp_config: /etc/dhcp/dhcpd.conf
foreman_proxy_dhcp_leases: /var/lib/dhcp/dhcpd.leases
```

## Dependencies

Additonal variables are required for configuring supporting services:  nginx, isc-dhcp server, tftp server, sqlite3, mariadb/mysql, postgresql, any services deployed utilizing the ansible-docker role.

These variables may be set in your group_vars


## Example Group Variables for Supporting Roles
```
# Foreman
www_domain: foreman.example.com
foreman_hostname: foreman
foreman_interface: 0.0.0.0
foreman_port: 3000
foreman_proxy_tftp: True
foreman_proxy_dhcp: True
foreman_db_name: foreman
foreman_db_username: foreman
foreman_db_password: "{{ vault_foreman_db_password }}"
foreman_admin_user: foreman
foreman_admin_password: "{{ vault_foreman_admin_password }}"
foreman_env: production

foreman_proxy_dhcp_subnets:
  - 10.0.10.0/255.255.255.0

# NGINX
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

# ISC-DHCP-SERVER
isc_dhcp_server_subnet:
  - netaddress: 10.0.10.0
    netmask: 255.255.255.0
    gateway: 10.0.10.1
    domain: "{{ www_domain | default(ansible_domain) }}"
    domain_search: "{{ www_domain | default(ansible_domain) }}"
    dns: 10.0.10.1
    range: 10.0.10.100 10.0.10.200
    vlanid: 0

# DOCKER
docker_containers:
  postgres:
    description: "Foreman Postgresql DB"
    image: postgres:11
    env:
      PUID: '0'
      PGID: '0'
      VERSION: 'latest'
      POSTGRES_USER: '{{ foreman_db_username }}'
      POSTGRES_PASSWORD: '{{ vault_foreman_db_password }}'
      POSTGRES_DB: '{{ foreman_db_name }}'
    ports:
      - '5432:5432'
    volumes:
      - '/etc/localtime:/etc/localtime:ro'
      - '/var/lib/postgresql/data:/var/lib/postgresql/data'

```

## Example Playbook
```
- name: "Deploy Foreman Server"
  hosts: foreman
  become: True
  remote_user: root
  tasks:

    - include_role:
        name: common

    - include_role:
        name: isc_dhcp_server
        public: yes
      when: foreman_proxy_dhcp

    - include_role:
        name: tftp
        public: yes
      when: foreman_proxy_tftp

    - include_role:
        name: nginx      

    - include_role:
        name: docker
        public: yes

    - include_role:
        name: foreman
        public: yes
      tags:
        - install
        - configure
        - foreman
        - smartproxy
```

## License

MIT

## Author Information

Created by Alan Janis
