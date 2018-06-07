
## Role Name: MySQL-Galera

An Ansible Role that installs MariaDB-Galera or Percona XtraDB Cluster on Debian/Ubuntu or RedHat/CentOS.

### Role Variables

#### `defaults/main.yml`

```
# Select whether to use Percona XtraDB Cluster or MariaDB + Galera
#mysql_provider: <percona|mariadb> 

mysql_provider: mariadb

# Enable vendor repo
enable_mysql_repo: True
install_mysql_from_pkgs: False

percona_release: 0.1-6
percona_version: 5.6
mariadb_release: 10.3
mariadb_version: 10.3.7

# Configure MySQL root user/password
mysql_root_username: root
mysql_root_password: "{{vault_mysql_root_password}}"

# Configure basic MySQL variables
mysql_daemon: mysql
mysql_daemon_user: mysql
mysql_config_dir: /etc/mysql/conf.d
mysql_socket: /var/run/mysqld/mysqld.sock
mysql_pid: /var/run/mysqld/mysqld.pid

# Set whether to use Galera multi-master clustering or non-clustered installation
galera_clustering: True

# Cluster Check Configuration
galera_clustercheck_enable: True
galera_clustercheck_username: clustercheck
galera_clustercheck_password: "{{vault_galera_clustercheck_password}}"

# Configure Galera WSREP
galera_wsrep_cluster_address: []
galera_bootstrap_node: []
galera_wsrep_on: On
galera_mysql_datadir: /var/lib/mysql
galera_wsrep_provider: /usr/lib/galera3/libgalera_smm.so
galera_wsrep_cluster_name: galera_cluster
galera_wsrep_slave_threads: 1
galera_wsrep_certify_nonPK: 1
galera_wsrep_max_ws_rows: 131072
galera_wsrep_max_ws_size: 1073741824
galera_wsrep_debug: 0
galera_wsrep_convert_lock_to_trx: 0
galera_wsrep_retry_autocommit: 1
galera_wsrep_auto_increment_control: 1
galera_wsrep_drupal_282555_workaround: 0
galera_wsrep_causal_reads: 0
galera_wsrep_notify_cmd: ""
galera_wsrep_sst_method: rsync

# Add MySQL databases
mysql_databases: []

# Add MySQL users
mysql_users: []
```
### Example Group Variables Configuration
#### `group_vars/openstack-project/vars`
```
mysql_provider: mariadb

mariadb_release: 10.3
mariadb_version: 10.3.7

enable_mysql_repo: False
install_mysql_from_pkgs: True

galera_clustering: True
galera_wsrep_cluster_address: ['10.0.10.1', '10.0.10.2', '10.0.10.3']
galera_bootstrap_node: "10.0.10.1"
galera_wsrep_provider: /usr/lib/galera/libgalera_smm.so
galera_clustercheck_enable: True
galera_clustercheck_username: clustercheck
galera_clustercheck_password: "{{vault_galera_clustercheck_password}}"

# MySQL Databases
mysql_databases:
  - name: keystone
    collation: utf8_general_ci
    encoding: utf8
  - name: glance
    collation: utf8_general_ci
    encoding: utf8
  - name: neutron
    collation: utf8_general_ci
    encoding: utf8
  - name: nova
    collation: utf8_general_ci
    encoding: utf8
  - name: nova_api
    collation: utf8_general_ci
    encoding: utf8
  - name: nova_cell0
    collation: utf8_general_ci
    encoding: utf8

# MySQL Users
mysql_users:
  - name: nova
    host: "%"
    password: "{{vault_mysql_openstack_password}}"
    priv: "nova.*:ALL/nova_api.*:ALL/nova_cell0.*:ALL"
  - name: keystone
    host: "%"
    password: "{{vault_mysql_openstack_password}}"
    priv: "keystone.*:ALL"
  - name: neutron
    host: "%"
    password: "{{vault_mysql_openstack_password}}"
    priv: "neutron.*:ALL"
  - name: glance
    host: "%"
    password: "{{vault_mysql_openstack_password}}"
    priv: "glance.*:ALL"
```
#### `group_vars/openstack-mysql/vault`
```
vault_mysql_nova_password: novapass
vault_mysql_keystone_password: keystonepass
vault_mysql_keystone_password: neutronpass
vault_mysql_glance_password: glancepass
vault_galera_clustercheck_password: clustercheckpass
```

### Example Playbook

I like to include my variables in `group_vars`, but you may wish to include them in your `playbook.yml` instead.

```
- name: Deploy MySQL-Galera
  hosts: openstack-mysql
  become: True
  gather_facts: True
  tasks:
    - include_role:
        name: mysql-galera
      vars:
        mysql_root_username: root
        mysql_root_password: root
        mysql_users:
          - name: nova
            host: "%"
            password: "{{vault_mysql_openstack_password}}"
            priv: "nova.*:ALL"
        mysql_databases:
          - name: nova
            collation: utf8_general_ci
            encoding: utf8    
        galera_clustercheck_enable: true
        galera_clustercheck_username: clustercheck
        galera_clustercheck_password: clustercheck
        galera_wsrep_cluster_address: ['10.0.10.1', '10.0.10.2', '10.0.10.3']
        galera_bootstrap_node: "10.0.10.1"
```

### Bootstrapping new cluser

```
ansible-playbook -e galera_bootstrapping=True mysql-galera.yml
```
