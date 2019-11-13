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

foreman_install_version: 1.23
foreman_plugins_version: 1.23
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
foreman_proxy_foreman_url: http://127.0.0.1

foreman_proxy_dhcp: True
foreman_proxy_dhcp_protocol: http
foreman_proxy_dhcp_server: 127.0.0.1
foreman_proxy_dhcp_omapi_port: 7911
foreman_proxy_dhcp_subnets: []
foreman_proxy_httpboot: True
foreman_proxy_tftp: True
foreman_proxy_tftp_server: 127.0.0.1
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

foreman_default_settings:
  - name: "default_pxe_item_global"
    value: "discovery"
  - name: "update_ip_from_built_request"
    value: "True"
  - name: "token_duration"
    value: "0"
  - name: "discovery_auto"
    value: "True"
  - name: "safemode_render"
    value: "False"
  - name: "discovery_location"
    value: "Default Location"
  - name: "discovery_organization"
    value: "Default Organization"

foreman_extra_settings: []

foreman_installation_mediums: []
foreman_operatingsystems: []
foreman_os_default_templates: []
foreman_hostgroups: []
foreman_global_parameters: []
```

### vars/debian.yml
```yaml
foreman_gpg_key_url: https://deb.theforeman.org/pubkey.gpg
foreman_repo_url: http://deb.theforeman.org/

python_pip_pkg: python-pip
foreman_python_pkgs:
  - apypie
  - requests
  - foreman-yml
  - foreman
  - PyYAML
  - setuptools

foreman_dependency_pkgs:
  - ca-certificates
  - python-httplib2
  - python-apt
  - curl
  - apt-transport-https
  - wget

foreman_pkgs:
  - foreman
  - foreman-libvirt
  - foreman-console
  - foreman-debug
  - foreman-journald
  - foreman-cli
  - ruby-foreman
  - ruby-hammer-cli
  - ruby-hammer-cli-foreman
  - ruby-hammer-cli-foreman-discovery
  - ruby-hammer-cli-foreman-remote-execution
  - ruby-powerbar
  - ruby-foreman-templates
  - ruby-foreman-tasks
  - ruby-foreman-remote-execution
  - ruby-foreman-discovery
  - ruby-foreman-bootdisk
  - ruby-foreman-dhcp-browser

foreman_proxy_pkgs:
  - foreman-proxy
  - foreman-proxy-journald
  - ruby-smart-proxy-discovery
  #- ruby-smart-proxy-remote-execution-ssh

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
foreman_db_sqlite_adapter_pkg: foreman-sqlite
foreman_db_mysql_adapter_pkg: foreman-mysql2
foreman_db_pgsql_adapter_pkg: foreman-postgresql
foreman_db_socket: /var/lib/mysql/mysql.sock

python_pip_pkg: python-pip

foreman_dependency_pkgs:
  - ca-certificates
  - python-httplib2
  - curl
  - wget
  - gcc
  - python-devel
  - python-setuptools

foreman_python_pkgs:
  - apypie
  - requests
  - foreman-yml
  - foreman
  - PyYAML

foreman_pkgs:
  - foreman
  - foreman-libvirt
  - foreman-console
  - foreman-debug
  - foreman-journald
  - foreman-cli
  - ruby-foreman
  - ruby-hammer-cli
  - ruby-hammer-cli-foreman
  - ruby-hammer-cli-foreman-discovery
  - ruby-hammer-cli-foreman-remote-execution
  - ruby-powerbar
  - ruby-foreman-templates
  - ruby-foreman-tasks
  - ruby-foreman-remote-execution
  - ruby-foreman-discovery
  - ruby-foreman-bootdisk
  - ruby-foreman-dhcp-browser

foreman_proxy_pkgs:
  - foreman-proxy
  - foreman-proxy-journald
  - ruby-smart-proxy-discovery
  #- ruby-smart-proxy-remote-execution-ssh


foreman_proxy_extra_repos_pkg: []

foreman_proxy_dhcp_config_dir: /etc/dhcp
foreman_proxy_dhcp_config: /etc/dhcp/dhcpd.conf
foreman_proxy_dhcp_leases: /var/lib/dhcp/dhcpd.leases

foreman_db_sqlite_adapter_pkg: foreman-sqlite3
foreman_db_mysql_adapter_pkg: foreman-mysql2
foreman_db_pgsql_adapter_pkg: foreman-postgresql

```


## Example Group Variables for Supporting Roles
```yaml
timezone: "America/Denver"

shared_storage: False
www_domain: home.prettybaked.com

foreman_install_version: 1.24
foreman_plugins_version: 1.24

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

foreman_extra_settings:
  - name: "root_pass"
    value: "{{ vault_foreman_admin_password }}"

foreman_proxy_dhcp_subnets:
  - name: "10.0.10.0/24"
    network: "10.0.10.0"
    mask: "255.255.255.0"
    gateway: "10.0.10.1"
    from_ip: "10.0.10.240"
    to_ip: "10.0.10.250"
    mtu: 9000
    domains:
    - "{{ www_domain }}"
    dns_primary: 10.0.10.1

foreman_installation_mediums:
  - name: "Ubuntu mirror"
    path: "http://archive.ubuntu.com/ubuntu"
    os_family: Debian

foreman_operatingsystems:
  - name: "Ubuntu"
    description: "Ubuntu 18.04"
    release_name: "bionic"
    family: "Debian"
    major: "18"
    minor: "4"
    provisioning_templates:
      - "Preseed default iPXE"
      - "Preseed default"
      - "Preseed default PXELinux"
      - "Preseed default finish"
    ptables:
      - "Preseed default"
    architectures:
      - x86_64
    media: "Ubuntu mirror"

foreman_os_default_templates:
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "finish"
    provisioning_template: "Preseed default finish"
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "provision"
    provisioning_template: "Preseed default"
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "PXELinux"
    provisioning_template: "Preseed default PXELinux"
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "iPXE"
    provisioning_template: "Preseed default iPXE"
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "user_data"
    provisioning_template: "Preseed default user data"

foreman_hostgroups:
  - name: "kvm"
    description: "KVM Hostgroup"
    architecture: x86_64
    operatingsystem: "Ubuntu 18.04"
    medium: "Ubuntu mirror"
    ptable: "Preseed default"
    domain: "{{ www_domain }}"
    subnet: "10.0.10.0/24"

foreman_hosts:
  - name: devbuild02
    domain: "{{ www_domain }}"
    architecture: "x86_64"
    hostgroup: "kvm"
    subnet: "10.0.10.0/24"
    ip: 10.0.10.241
    operatingsystem: "Ubuntu 18.04"
    media: "Ubuntu mirror"
    ptable: "Preseed default"
    mac_address: "52:54:00:7a:d0:c4"
  - name: devbuild05
    domain: "{{ www_domain }}"
    architecture: "x86_64"
    hostgroup: "kvm"
    operatingsystem: "Ubuntu 18.04"
    media: "Ubuntu mirror"
    ptable: "Preseed default"
    mac_address: "52:54:00:5b:ce:84"
    subnet: "10.0.10.0/24"
    ip: 10.0.10.243
  - name: devbuild06
    domain: "{{ www_domain }}"
    architecture: "x86_64"
    hostgroup: "kvm"
    operatingsystem: "Ubuntu 18.04"
    media: "Ubuntu mirror"
    ptable: "Preseed default"
    mac_address: "52:54:00:dd:fa:3a"
    subnet: "10.0.10.0/24"
    ip: 10.0.10.244

nginx_backends:
  - service: foreman
    servers:
      - localhost:3000
  - service: awx
    servers:
      - localhost:8052

nginx_vhosts:
  - servername: "{{ foreman_hostname | default(ansible_hostname) + '.' + www_domain | default(ansible_domain) }}"
    serveralias: "{{ ansible_default_ipv4.address }} {{ ansible_fqdn }}"
    serverlisten: 80
    extra_parameters: |
      error_page 404 /404.html;
      error_page 500 502 503 504 /50x.html;
    locations:
      - name: /
        proxy: True
        backend: foreman
      - name: 404.html
        docroot: "/usr/share/nginx/html"
      - name: 50x.html
        docroot: "/usr/share/nginx/html"
  - servername: "awx.home.prettybaked.com"
    serveralias: "awx"
    serverlisten: 80
    locations:
      - name: /
        proxy: True
        backend: awx

nginx_vhosts_ssl: []

docker_networks:
  docker_awx:
    ipam_config:
      - subnet: 172.3.27.0/24

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

  awx_postgres:
    description: AWX Postgres DB
    image: postgres:10
    hostname: postgres
    networks:
      - name: docker_awx
        aliases: postgres
    volumes:
      - "{{ postgres_data_dir }}/10/data/:/var/lib/postgresql/data/pgdata:Z"
    env:
      POSTGRES_USER: "{{ pg_username }}"
      POSTGRES_PASSWORD: "{{ pg_password }}"
      POSTGRES_DB: "{{ pg_database }}"
      PGDATA: "/var/lib/postgresql/data/pgdata"
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

  awx_rabbitmq:
    description: AWX RabbitMQ Service
    image: "{{ rabbitmq_image }}"
    volumes:
      - '/etc/localtime:/etc/localtime:ro'
    networks:
      - name: docker_awx
        aliases: rabbitmq
    hostname: rabbitmq
    env:
      RABBITMQ_DEFAULT_VHOST: "{{ rabbitmq_default_vhost }}"
      RABBITMQ_DEFAULT_USER: "{{ rabbitmq_user }}"
      RABBITMQ_DEFAULT_PASS: "{{ rabbitmq_password | quote }}"
      RABBITMQ_ERLANG_COOKIE: "{{ rabbitmq_erlang_cookie }}"
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

  awx_memcached:
    description: AWX Memcached Service
    image: "{{ memcached_image }}:{{ memcached_version }}"
    hostname: "{{ memcached_host }}"
    networks:
      - name: docker_awx
        aliases: "{{ memcached_host }}"
    volumes:
      - '/etc/localtime:/etc/localtime:ro'
    env:
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

  awx:
    description: AWX Service
    image: "{{ dockerhub_base }}/awx_task:{{ dockerhub_version }}"
    hostname: "{{ awx_task_hostname }}"
    depends_on:
      - docker-awx_memcached.service
      - docker-awx_rabbitmq.service
      - docker-awx_postgres.service
    networks:
      - name: docker_awx
        aliases:
          - task
          - awx
        links:
          - awxweb:awx_web
          - awxweb:web
    user: root
    volumes:
      - "{{ docker_compose_dir }}/SECRET_KEY:/etc/tower/SECRET_KEY"
      - "{{ docker_compose_dir }}/environment.sh:/etc/tower/conf.d/environment.sh"
      - "{{ docker_compose_dir }}/credentials.py:/etc/tower/conf.d/credentials.py"
      - "{{ project_data_dir }}:/var/lib/awx/projects:rw"
    env:
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

  awxweb:
    description: AWX Web Application
    image: "{{ dockerhub_base }}/awx_web:{{ dockerhub_version }}"
    depends_on:
      - docker-awx_memcached.service
      - docker-awx_rabbitmq.service
      - docker-awx_postgres.service
    ports:
      - "{{ host_port }}:8052"
    networks:
      - name: docker_awx
        aliases:
          - web
          - awxweb
        links:
          - awx:awx_task
          - awx:task
    hostname: "{{ awx_web_hostname }}"
    user: root
    volumes:
      - "{{ docker_compose_dir }}/SECRET_KEY:/etc/tower/SECRET_KEY"
      - "{{ docker_compose_dir }}/environment.sh:/etc/tower/conf.d/environment.sh"
      - "{{ docker_compose_dir }}/credentials.py:/etc/tower/conf.d/credentials.py"
      - "{{ docker_compose_dir }}/nginx.conf:/etc/nginx/nginx.conf:ro"
      - "{{ project_data_dir }}:/var/lib/awx/projects:rw"
    env:
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

rabbitmq_version: "3.7.4"
rabbitmq_image: "ansible/awx_rabbitmq:{{rabbitmq_version}}"
rabbitmq_default_vhost: "awx"
rabbitmq_erlang_cookie: "{{ vault_rabbitmq_erlang_cookie }}"
rabbitmq_host: "rabbitmq"
rabbitmq_port: "5672"
rabbitmq_user: "guest"
rabbitmq_password: "{{ vault_rabbitmq_password }}"

postgresql_version: "10"
postgresql_image: "postgres:{{postgresql_version}}"


memcached_image: "memcached"
memcached_version: "alpine"
memcached_host: "memcached"
memcached_port: "11211"

dockerhub_base: ansible
dockerhub_version: 9.0.0
create_preload_data: True

project_data_dir: /var/lib/awx
postgres_data_dir: /var/lib/pgawx
host_port: 8080

docker_compose_dir: /var/lib/awxcompose
pg_username: awx
pg_password: "{{ vault_pg_password }}"
pg_admin_password: "{{ vault_pg_admin_password }}"
pg_database: awx
pg_port: 5432

secret_key: "{{ vault_secret_key }}"
admin_user: admin
admin_password: "{{ vault_awx_admin_password }}"

awx_web_hostname: awxweb
awx_task_hostname: awx


sshd_permit_root_login: 'yes'

ssh_users:
  ajanis:
    password: "{{vault_ajanis_pw}}"
    cn: "Alan Janis"
    givenname: "Alan"
    sn: "Janis"
    mail: "alan.janis@gmail.com"
    shell: /bin/zsh
    gecos: ajanis
    uid: 1043
    gid: 1042
    pubkey: "{{ vault_id_rsa_pubkey }}"
    state: present
```

**NOTE:** If ```foreman_proxy_dhcp_subnets``` is defined, it will be used by the isc-dhcp-server role to configure dhcpd.conf, so you **do not** need to define your DHCP subnets again as ```isc_dhcp_server_subnets```

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