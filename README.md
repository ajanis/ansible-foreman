# Ansible-Foreman

<!-- MarkdownTOC -->

- Dependencies
- Requirements
- Role Variables
  - defaults/main.yml
  - vars/debian.yml
  - vars/redhat.yml
- Example Group Variables for Supporting Roles
- Example Playbook
- License
- Author Information

<!-- /MarkdownTOC -->


**This role is part of the [ITC Automated Build Pods Project][]**

Deploy and configure the Foreman server with smart-proxy managed TFTP and DHCP services.

## Dependencies

The role itself has no dependencies and can be used to install a basic foreman server.  However it is intended to be used as part of the [ITC Automated Build Pods Project][] which requires additional Ansible roles and configurations.

## Requirements

 * The initial configuration of the Foreman Smart-Proxy is handled using [foreman-yml][].  This role supports configuration of DHCP, deployed via [Ansible ISC DHCP Server Role][] and TFTP, deployed via [Ansible TFTP Role][].
 * Extended configuration of the Foreman server is handled via [Ansible Foreman Modules][] and should be handled via ```group_vars```
 * If you wish to use Postgres or MySQL for the Foreman Database backend, you can deploy it in a container using this [Ansible Docker Role][].  The default database is Postgres
 * Proxied requests to the Foreman server and API are handled by NGINX, which is deployed using this [Ansible NGINX Role][].

## Role Variables

### defaults/main.yml
```yaml
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

foreman_proxy_dhcp_subnets:
  - name: "10.0.10.0/24"
    network: "10.0.10.0"
    mask: "255.255.255.0"
    gateway: "10.0.10.1"
    from_ip: "10.0.10.240"
    to_ip: "10.0.10.250"
    vlanid:
    mtu: 9000
    domains:
    - "{{ www_domain }}"
    dns_primary: 10.0.10.1
    dns_secondary:

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
```yaml
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
```yaml
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


## Example Group Variables for Supporting Roles
```yaml
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
```
**NOTE:** If ```foreman_proxy_dhcp_subnets``` is defined, it will be used by the isc-dhcp-server role to configure dhcpd.conf, so you **do not** need to define your DHCP subnets again as ```isc_dhcp_server_subnets```

```yaml
# isc_dhcp_server_subnets:

foreman_proxy_dhcp_subnets:
  - name: "10.0.10.0/24"
    network: "10.0.10.0"
    mask: "255.255.255.0"
    gateway: "10.0.10.1"
    from_ip: "10.0.10.240"
    to_ip: "10.0.10.250"
    vlanid:
    mtu: 9000
    domains:
    - "{{ www_domain }}"
    dns_primary: 10.0.10.1
    dns_secondary:

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
```yaml
- name: "Deploy Foreman Server"
  hosts: all
  remote_user: root
  tasks:

    - setup:
    - include_role:
        name: common
      tags:
        - common

    - include_role:
        name: isc_dhcp_server
        public: yes
        apply:
          tags:
            - dhcp
      when: foreman_proxy_dhcp
      tags:
        - dhcp

    - include_role:
        name: tftp
        public: yes
        apply:
          tags:
            - tftp
      when: foreman_proxy_tftp
      tags:
        - tftp

    - include_role:
        name: nginx
        public: yes
        apply:
          tags:
            - nginx
      tags:
        - nginx

    - include_role:
        name: awx
        public: yes
        apply:
          tags:
            - awx
      tags:
        - awx

    - include_role:
        name: docker
        public: yes
        apply:
          tags:
            - docker
      tags:
        - docker

    - include_role:
        name: awx
        tasks_from: update_ca.yml
        public: yes
        apply:
          tags:
            - awx
      tags:
        - awx

    - include_role:
        name: foreman
        public: yes
      tags:
        - install
        - configure
        - foreman
        - smartproxy

    - include_role:
        name: foreman
        public: yes
        tasks_from: customize_config.yml
        apply:
          tags:
            - customize
      tags:
        - customize

    - include_role:
        name: foreman
        public: yes
        tasks_from: host-build.yml
      tags:
        - hostcreate
        - hostcleanup
```

## License

MIT

## Author Information

Created by Alan Janis

[Ansible Foreman Modules]: https://github.com/theforeman/foreman-ansible-modules
[Ansible Docker Role]: https://github.com/ajanis/ansible-docker.git
[foreman-yml]: https://pypi.org/project/foreman-yml
[ansible tftp role]:  https://github.com/ajanis/ansible-tftp.git
[ansible nginx role]:  https://github.com/ajanis/ansible-nginx.git
[itc automated build pods project]:  https://github.com/ajanis/itc-build-pods.git
[ansible isc dhcp server role]:  https://github.com/ajanis/ansible-isc-dhcp-server.git