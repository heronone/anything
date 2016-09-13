# 0 Overview
First of all,prepare an environment to install OpenStack. This guidance uses the minimal architecture, that is to say, a **controller node** and a **compute node**.After the configurations of the environment, install services for OpenStack.

The main services those are necessary to launch a basic instance are **identity service**, **image service**, **compute service**, and **networking service**, or generally called **keystone**, **glance**, **nova** and **neutron**.

## 0.1 Command prompts
1. Any user can run this command:
```
$ command
```

2. Use `root` user to run this command:
```
# command
```

***

# 1 Configure the environment
## 1.1 Networking
Configure a management network interface and a provider network interface.
### 1.1.1 Controller Node
1. Edit `/etc/network/interfaces` like this, replace the address as the actual one you need. The managemment interface should be able to connect to Internet:
```
#management network interfaces
auto em1
iface em1 inet static
        address 192.168.1.11
        netmask 255.255.255.0
        gateway 192.168.1.1
        dns-nameservers 8.8.8.8
#############################
#provider network interfaces
auto em2
iface em2 inet manual
        up ip link set dev $IFACE up
        down ip link set dev $IFACE down
```

2. Edit `/etc/hosts` like this, remove the line `127.0.1.1` if it exists:
```
127.0.0.1 localhost controller
192.168.1.11 controller
192.168.1.31 compute
```

3. Reboot the system to activate the changes.

### 1.1.2 Compute Node
1. Edit `/etc/network/interfaces` like this, replace the address as the actual one you need. The managemment interface should be able to connect to Internet:
```
#management network interfaces
auto em1
iface em1 inet static
        address 192.168.1.31
        netmask 255.255.255.0
        gateway 192.168.1.1
        dns-nameservers 8.8.8.8
#############################
#provider network interfaces
auto em2
iface em2 inet manual
        up ip link set dev $IFACE up
        down ip link set dev $IFACE down
```

2. Edit `/etc/hosts` like this, remove the line `127.0.1.1` if it exists:
```
127.0.0.1 localhost
192.168.1.11 controller
192.168.1.31 compute
```
3. Reboot the system to activate the changes.

## 1.2 Network Time Protocol
Here choose NTP, you can also choose Chrony if you like.

### 1.2.1 Controller Node
1. Install package:
```
# apt-get install ntp
```
if `/var/lib/net/ntp.conf.dhcp` exist, delete it.

2. Restart service:
```
# service ntp restart
```

### 1.2.2 Compute Node
1. Install package:
```
# apt-get install ntp
```
if `/var/lib/net/ntp.conf.dhcp` exist, delete it.

2. Edit `/etc/ntp.conf` like this, replace 192.168.1.11 to your actual controller ip:
```
#comment out all other server key
server 192.168.1.11 iburst
```

3. Restart service:
```
# service ntp restart
```

### 1.2.3 (Optional) Verify Operation
1. In compute node:
```
 # ntpq -c peers
```
you should see something like this:
```
remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*controller      YOU_NTP_SERVER_IP    3 u   61 1024  377    0.315   -0.256   0.228
```

2. then:
```
# ntpq -c assoc
```
 you should see something like this:
```
ind assid status  conf reach auth condition  last_event cnt
===========================================================
  1 12003  965a   yes   yes  none  sys.peer    sys_peer  5
```

## 1.3 Install Openstack Packages
In both controller node and compute node,do:

```
# apt-get install software-properties-common
```

```
# apt-get install python-openstackclient
```

## 1.4 Install SQL database
1. Install packages:
```
# apt-get install mariadb-server python-pymysql
```

2. Create and edit the `/etc/mysql/conf.d/openstack.cnf` file:
```
[mysqld]
...
bind-address = 192.168.1.11
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

3. Restart the sql service:
```
# service mysql restart
```

4. (Optional) Secure the database service by this command:
```
# mysql_secure_installation
```

## 1.5 Install Message Queue
1. Install package:
```
# apt-get install rabbitmq-server
```

2.  Add the `openstack` user, replace `RABBIT_PASS` as the password you need:
```
# rabbitmqctl add_user openstack RABBIT_PASS
```

3. Permit configuration, write, and read access for the `openstack` user:
```
# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

## 1.6 Install Memcached
1. Inatall packages:
```
# apt-get install memcached python-memcache
```

2. Edit the `/etc/memcached.conf` file:
```
-l 192.168.1.11
```

3. Restart the Memcached service:
```
# service memcached restart
```

***

# 2 Identity service (keystone)

## 2.1 Create a database
Connect to the database server as the `root` user:
```
$ mysql -u root -p
```

Create the `keystone` database:
```
CREATE DATABASE keystone;
```

Grant proper access to the `keystone` database, replace `KEYSTONE_DBPASS` with a suitable password:
```
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
```

Exit the database access client.

## 2.2 Install and configure keystone
1. install packages:
```
# apt-get install keystone apache2 libapache2-mod-wsgi
```
2. Generate a random value to use as the `ADMIN_TOKEN` during initial configuration:
```
$ openssl rand -hex 10
```

3. Edit the `/etc/keystone/keystone.conf` file like this:
```
[DEFAULT]
...
admin_token = ADMIN_TOKEN
##########################
[database]
...
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
##########################
[token]
...
provider = fernet
```

4. Populate the Identity service database:
```
# su -s /bin/sh -c "keystone-manage db_sync" keystone
```
May be something wrong with this command, see http://blog.csdn.net/zhujie_hades/article/details/52104116 when you get error.(I am not sure if it is helpful for you, but I solved my problem referring to this site.Thanks a lot!)

5. Initialize Fernet keys:
```
# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```

## 2.3 Configure Apache2
1. Edit `/etc/apache2/apache2.conf`, add this:
```
ServerName controller
```

2. Create and edit `/etc/apache2/sites-available/wsgi-keystone.conf`, add this:
```
Listen 5000
Listen 35357
############################
<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
############################
<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
```

3. Enable the Identity service virtual hosts:
```
# ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
```

4. Restart the service:
```
# service apache2 restart
```

## 2.4 Create the service entity and API endpoints
1. Configure environment variables:
```
$ export OS_TOKEN=ADMIN_TOKEN
$ export OS_URL=http://controller:35357/v3
$ export OS_IDENTITY_API_VERSION=3
```

2. then:
```
$ openstack service create --name keystone --description "OpenStack Identity" identity
$ openstack endpoint create --region RegionOne identity public http://controller:5000/v3
$ openstack endpoint create --region RegionOne identity internal http://controller:5000/v3
$ openstack endpoint create --region RegionOne identity admin http://controller:35357/v3
```

## 2.5 Create a domain, projects, users, and roles
```
$ openstack domain create --description "Default Domain" default
$ openstack project create --domain default --description "Admin Project" admin
$ openstack user create --domain default --password-prompt admin
$ openstack role create admin
$ openstack role add --project admin --user admin admin
$ openstack project create --domain default --description "Service Project" service
$ openstack project create --domain default --description "Demo Project" demo
$ openstack user create --domain default --password-prompt demo
$ openstack role create user
$ openstack role add --project demo --user demo user
```

## 2.6 Verify operation
1. disable the temporary authentication token mechanism:Edit `/etc/keystone/keystone-paste.ini` and remove `admin_token_auth` from the `[pipeline:public_api]`, `[pipeline:admin_api]`, and `[pipeline:api_v3]` sections.

2. Unset environment variables:
```
$ unset OS_TOKEN OS_URL
```

3. As the `admin` user, request an authentication token:
```
$ openstack --os-auth-url http://controller:35357/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name admin --os-username admin token issue
```

4. As the `demo` user, request an authentication token:
```
$ openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name demo --os-username demo token issue
```

## 2.7 Create OpenStack client environment scripts
1. Creat and edit the `admin-openrc` file and add the following content:
```
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
Replace `ADMIN_PASS` with the password you set for the `admin` user in the Identity service.

2. Creat and edit the `demo-openrc` file and add the following content:
```
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
Replace `DEMO_PASS` with the password you set for the `demo` user in the Identity service.

3. While using the scripts, use command like this:
```
$ . admin-openrc
```

## 2.8 How to delete/remove/uninstall identity service
1. Remove all the packages you have installed:
```
# apt-get purge *
```

2. Connect to database and delete the `keystone` database:
```
$ mysql -u root -p
```
then
```
drop database keystone;
```

***

# 3. Image service (glance)
Please check the openstack official sites.

***

# 4. Compute service (nova)
Please check the openstack official sites.

***

# 5. Networking Service (neutron)
Please check the openstack official sites.

***

# 6. Launch an instance

## 6.1 Create virtual networks

### 6.1.1 Create provider network

1. On the controller node,source the `admin` credentials:
```
$ . admin-openrc
```

2. Create the network:
```
$ neutron net-create --shared --provider:physical_network provider --provider:network_type flat provider
```

3. Create a subnet on provider network:
```
$ neutron subnet-create --name provider \
  --allocation-pool start=START_IP_ADDRESS,end=END_IP_ADDRESS \
  --dns-nameserver DNS_RESOLVER --gateway PROVIDER_NETWORK_GATEWAY \
  provider PROVIDER_NETWORK_CIDR
```
**This subnet should be in the same lan with your management interfaces.
This subnet should be in the same lan with your management interfaces.
This subnet should be in the same lan with your management interfaces.**
For example:
```
$ neutron subnet-create --name provider \
  --allocation-pool start=192.168.1.101,end=192.168.1.200 \
  --dns-nameserver 8.8.8.8 --gateway 192.168.1.1 \
  provider 192.168.1.0/24
```

### 6.1.2 Create self-service network
Please check the openstack official sites.

## 6.2 and more and more after
Please check the openstack official sites.

***
