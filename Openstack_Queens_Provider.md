<h1> Cài đặt Openstack Queens theo docs mô hình Provider networks </h1>
<h2> I. Cài đặt cơ bản </h2>
<h4>1. Chuẩn bị môi trường </h4>
<h4><li> Mô hình mạng </li></h4>
<img src="https://github.com/anhict/images/blob/master/net.PNG?raw=true">
<li> Sử dụng 2 node đã cài sẵn Ubuntu phiên bản 16.04.</li>
<h4> <li> 2. Cài đặt trên controller node </li></h4>
<p> Lưu ý: </p>
<ul>
<p><li> Đăng nhập với quyền root trên tất cả các bước cài đặt.</li> </p>
<p><li> Các thao tác sửa file trong hướng dẫn này sử dụng lệnh vi hoặc vim</li> </p>
<p><li> Password thống nhất cho tất cả các dịch vụ là Welcome123</li> </p>
</ul>
<h4> 2.1 Cài đặt các thành phần cơ bản </h4>
<h6> 2.1.1 Thiết lập và cài đặt các gói cơ bản</h6>
<li> Chạy lệnh để cập nhật các gói phần mềm </li>
<pre>  apt-get -y update </pre> 
<li> Thiết lập địa chỉ IP</li>
<li>Dùng lệnh <code> vi </code>để sửa file <code> /etc/network/interfaces</code> với nội dung như sau.</li>

<pre>
 # Interface EXT
 auto ens33
 iface ens33 inet static
 	address 192.168.239.162
 	netmask 255.255.255.0
 	gateway 192.168.239.2
 	dns-nameservers 8.8.8.8
</pre>


<li>Khởi động lại card mạng sau khi thiết lập IP tĩnh</li>

<pre>ifdown –a && ifup –a </pre>
<li>Cấu hình hostname</li>

Sử dụng lệnh vi /etc/hosts sử file theo nội dụng sau trên controller node và compute1 node :
<pre>
 127.0.0.1      localhost controller
 192.168.239.162    controller
 192.168.239.163    compute1
 </pre>


<li> Kiểm tra ping trên controller node và compute1 node</li>
<pre>
root@compute1:~# ping google.com
PING google.com (216.58.199.14) 56(84) bytes of data.
64 bytes from hkg12s02-in-f14.1e100.net (216.58.199.14): icmp_seq=1 ttl=128 time=37.1 ms
64 bytes from hkg12s02-in-f14.1e100.net (216.58.199.14): icmp_seq=2 ttl=128 time=39.3 ms
64 bytes from hkg12s02-in-f14.1e100.net (216.58.199.14): icmp_seq=3 ttl=128 time=36.7 ms
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 36.706/37.728/39.312/1.135 ms
root@compute1:~#
</pre>
<pre>
root@controller:~# ping google.com
PING google.com (172.217.25.14) 56(84) bytes of data.
64 bytes from hkg07s24-in-f14.1e100.net (172.217.25.14): icmp_seq=1 ttl=128 time=32.1 ms
64 bytes from hkg07s24-in-f14.1e100.net (172.217.25.14): icmp_seq=2 ttl=128 time=31.1 ms
64 bytes from hkg07s24-in-f14.1e100.net (172.217.25.14): icmp_seq=3 ttl=128 time=32.5 ms
64 bytes from hkg07s24-in-f14.1e100.net (172.217.25.14): icmp_seq=4 ttl=128 time=28.0 ms
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3036ms
rtt min/avg/max/mdev = 28.001/30.961/32.524/1.785 ms
root@controller:~#
</pre>

<h6> 2.1.2 Cài đặt Network Time Protocol (NTP) </h6>
<p><li> Cài đặt các packages </li></p>
<pre>  apt-get -y install chrony </pre>

Sửa file vi /etc/chrony/chrony.conf 
Comment dòng 
<pre> pool 2.debian.pool.ntp.org offline iburst </pre>
Thay thế bằng dòng :
<pre>server controller iburst </pre>
Thêm dòng :
<pre> allow 192.168.239.0/24 </pre>
<li> Khởi động lại dịch vụ NTP </li>
<pre> service chrony restart </pre>
<li> Chạy lệnh kiểm tra : </li>
<pre> root@controller:~# chronyc sources </pre>

<img src="https://github.com/anhict/images/blob/master/26.PNG?raw=true">

<h6> 2.1.2 Cài đặt OpenStack packages trên tất cả các node </h6>

<pre> 
# apt install software-properties-common
# add-apt-repository cloud-archive:queens 
</pre><li> Cập nhật các gói phần mềm trên tất cả các node </li>
<pre> 
# apt update && apt dist-upgrade
</pre>

<li> Cài đặt OpenStack client: </li>
<pre>
# apt install python-openstackclient
</pre>
<li> Khởi động lại: </li>
<pre>
init 6
</pre>
<h6> 2.1.4 Cài đặt SQL database </h6>
<li> Cài đặt các packages </li>
<pre>apt install mariadb-server python-pymysql
</pre><li>Tạo file /etc/mysql/mariadb.conf.d/99-openstack.cnf với nội dung sau :
<pre> vi /etc/mysql/mariadb.conf.d/99-openstack.cnf
</pre>
<pre>
[mysqld]
bind-address = 0.0.0.0

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
</pre><li> Khởi động lại dịch vụ SQL </li>
<pre> 
# service mysql restart
</pre><li> Thiết lập cơ bản cho MariaDB </li>
<pre> 
mysql_secure_installation
</pre>

<pre>  mysql -u root -pWelcome123
</pre>

<pre>  
 root@controller:~# mysql -u root -pWelcome123
 Enter password:
 Welcome to the MariaDB monitor.  Commands end with ; or \g.
 Your MariaDB connection id is 29
 Server version: 5.5.47-MariaDB-1ubuntu0.14.04.1 (Ubuntu)

 Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.

 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

 MariaDB [(none)]>
 MariaDB [(none)]>
 MariaDB [(none)]> exit;
</pre>

<h6> 2.1.5 Cài đặt RabbitMQ </h6>
<li> Cài đặt packages </li>
<pre>  # apt install rabbitmq-server</pre>
<li> Gán quyền read, write cho tài khoản openstack trong RabbitMQ </li>

<pre>  # rabbitmqctl set_permissions openstack ".*" ".*" ".*"
</pre>

<h6> 2.1.5 Cài đặt  Memcached </h6>
<pre>  # apt install memcached python-memcache
</pre>
Sửa file /etc/memcached.conf
<pre>  -l 192.168.239.162
</pre>
<li> Khởi động lại dịch vụ </li>
<pre>  # service memcached restart
</pre>
<h6> 2.1.6 Cài đặt  Etcd </h6>
<li>Tạo etcd user:</li>
<pre># groupadd --system etcd
# useradd --home-dir "/var/lib/etcd" \
      --system \
      --shell /bin/false \
      -g etcd \
      etcd </pre>
 <li>Tạo các thư mục cần thiết: </li>
 <pre>
# mkdir -p /etc/etcd
# chown etcd:etcd /etc/etcd
# mkdir -p /var/lib/etcd
# chown etcd:etcd /var/lib/etcd
</pre>
 <li>Download and install the etcd tarball:  </li>
 <pre>
 # ETCD_VER=v3.2.7
# rm -rf /tmp/etcd && mkdir -p /tmp/etcd
# curl -L https://github.com/coreos/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
# tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd --strip-components=1
# cp /tmp/etcd/etcd /usr/bin/etcd
# cp /tmp/etcd/etcdctl /usr/bin/etcdctl
</pre>

<li>Tạo file /etc/etcd/etcd.conf.yml </li>
<pre>
name: controller
data-dir: /var/lib/etcd
initial-cluster-state: 'new'
initial-cluster-token: 'etcd-cluster-01'
initial-cluster: controller=http://192.168.239.162:2380
initial-advertise-peer-urls: http://192.168.239.162:2380
advertise-client-urls: http://192.168.239.162:2379
listen-peer-urls: http://0.0.0.0:2380
listen-client-urls: http://192.168.239.162:2379
</pre>
<li> Tạo file /lib/systemd/system/etcd.service </li>
<pre>
[Unit]
After=network.target
Description=etcd - highly-available key value store

[Service]
LimitNOFILE=65536
Restart=on-failure
Type=notify
ExecStart=/usr/bin/etcd --config-file /etc/etcd/etcd.conf.yml
User=etcd

[Install]
WantedBy=multi-user.target
</pre>
<li> Khởi động lại dịch vụ </li>
<pre>
# systemctl enable etcd
# systemctl start etcd
</pre>


<h4> 3. Cài đặt Keystone </h4>
<h6> 3.1 Cài đặt và cấu hình cho keysonte </h6>
<h6> 3.1.1. Tạo database cài đặt các gói và cấu hình keystone </h6>
<li> Đăng nhập vào MariaDB </li>
<pre>mysql -u root -pWelcome123</pre>
<li> Tạo user, database cho keystone </li>
<pre>
MariaDB [(none)]> CREATE DATABASE keystone;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'Welcome123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'Welcome123';
</pre>
<li> Cài đặt các packages </li>
<pre># apt install keystone  apache2 libapache2-mod-wsgi </pre>
<li> Dùng lệnh vi sửa file  /etc/keystone/keystone.conf với nội dung sau : </li>
<pre>
[database]
connection = mysql+pymysql://keystone:Welcome123@controller/keystone
[token]
provider = fernet
</pre>
<li> Đồng bộ database cho keystone </li>
<pre> # su -s /bin/sh -c "keystone-manage db_sync" keystone </pre>
<li> Khởi tạo kho lưu trữ khóa Fernet:</li>
<pre>
# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
</pre>
<li> Khởi động dịch vụ Identity </li>
<pre>
# keystone-manage bootstrap --bootstrap-password Welcome123 \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
 </pre>
 <li> Sử dụng lệnh vi sửa file /etc/apache2/apache2.conf thêm vào dòng: </li>
 <pre> ServerName controller </pre>
 
 <li> Khởi động lại dịch vụ : </li>
 <pre># service apache2 restart </pre>
 <li> Cấu hình tài khoản </li>
 <pre>
$ export OS_USERNAME=admin
$ export OS_PASSWORD=Welcome123
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://controller:5000/v3
$ export OS_IDENTITY_API_VERSION=3
</pre>

<li> Tạo  domain, projects, users, and roles </li>
<pre>openstack domain create --description "An Example Domain" example</pre>
<pre>openstack project create --domain default \
  --description "Service Project" service </pre>
<pre>openstack project create --domain default \
  --description "Demo Project" demo </pre>
<pre>openstack user create --domain default \
  --password-prompt demo </pre>
<pre>openstack role create user </pre>
<pre>openstack role add --project demo --user demo user </pre>

<li> Verify </li>
- Bỏ thiết lập trong biến môi trường của OS_TOKEN và OS_URL bằng lệnh
<pre>$ unset OS_AUTH_URL </pre>
-  Là người dùng quản trị, yêu cầu mã thông báo xác thực:


<pre>$ openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue </pre>
<pre>$ openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name demo --os-username demo token issue </pre>
  
  <li> Tạo file admin-openrc với nội dung sau :</li>
<pre>export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2 </pre>
 <li> Tạo file demo-openrc với nội dung sau :</li>
 <pre>export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2</pre>


<li> Chạy script admin-openrc </li>
<pre> $ . admin-openrc </pre>
<li> Gõ lệnh dưới để kiểm tra biến môi trường ở trên đã chính xác hay chưa </li>
<pre> $ openstack token issue </pre>
<h4> 4. Cài đặt Glance </h4>
Glance là dịch vụ cung cấp các image (các hệ điều hành đã được đóng gói sẵn), các image này sử dụng theo cơ chế template để tạo ra các máy ảo. )

Lưu ý: Thư mục chứa các file images trong hướng dẫn này là /var/lib/glance/images/

Glance có các thành phần sau:
<ul>
<li>glance-api:</li>
<li>glance-registry:</li>
<li>Database:</li>
<li>Storage repository for image file</li>
<li>Metadata definition service</li>
</ul>
<h6> 4.1. Tạo database và endpoint cho glance </h6>
<li> Đăng nhập vào mysql <li>
<pre> mysql -uroot -pWelcome123 </pre>
<pre>
 MariaDB [(none)]> CREATE DATABASE glance;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
IDENTIFIED BY 'Welcome123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
IDENTIFIED BY 'Welcome123';
</pre>

<h6> 4.1.2. Cấu hình xác thực cho dịch vụ glance </h6>
<li>Thiết lập biến môi trường:</li>
<pre>$ . admin-openrc</pre>
<li>Tạo user Glance </li>
<pre>$ openstack user create --domain default --password-prompt glance</pre>
<li>Add the admin role to the glance user and service project:</li>
<pre>$ openstack role add --project service --user glance admin</pre>
</li>Tạo glance service</li>
<pre>$ openstack service create --name glance \
  --description "OpenStack Image" image
</pre>
<li>Tạo các endpoint cho dịch vụ glance</li>
<pre>
$ openstack endpoint create --region RegionOne \
  image public http://controller:9292
$ openstack endpoint create --region RegionOne \
  image internal http://controller:9292
$ openstack endpoint create --region RegionOne \
  image admin http://controller:9292
</pre>

<h6>4.1.3. Cài đặt các gói và cấu hình cho dịch vụ glance</h6>
<pre># apt install glance </pre>
<li>Chỉnh sửa file /etc/glance/glance-api.conf </li>
<pre>
[database]
connection = mysql+pymysql://glance:Welcome123@controller/glance
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
[paste_deploy]
flavor = keystone
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
</pre>
<li>Sửa file /etc/glance/glance-registry.conf </li>
<pre>
[database]
connection = mysql+pymysql://glance:Welcome123@controller/glance
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = Welcome123
[paste_deploy]
flavor = keystone
</pre>
<li> Đồng bộ database cho glance </li>
<pre># su -s /bin/sh -c "glance-manage db_sync" glance</pre>
<li>Khởi động lại dịch vụ Glance</li>
<pre># service glance-registry restart
# service glance-api restart
</pre>


<li> Thiết lập biến môi trường :</li>
<pre>$ . admin-openrc</pre>
<li> Download the source image: </li>
<pre>$ wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img</pre>
<li>Upload file image vừa tải về </li>
<pre>
openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
  </pre>
  <li>Kiểm tra</li>
  <pre> openstack image list
 </pre>
 <pre>
 root@controller:~# openstack image list
 +--------------------------------------+--------+--------+
 | ID                                   | Name   | Status |
 +--------------------------------------+--------+--------+
 | 19d53e24-2985-4f75-bd63-7568a5f2f10f | cirros | active |
 +--------------------------------------+--------+--------+
 root@controller:~#
 </pre>
 <h4>Cài đặt NOVA (Compute service)</h4>
<h6>5.2. Cài đặt và cấu hình nova</h6>
<li>Đăng nhập vào database với quyền root</li>
<pre>mysql -uroot -pWelcome123</pre>
<li>Tạo database</li>
<pre>
MariaDB [(none)]> CREATE DATABASE nova_api;
MariaDB [(none)]> CREATE DATABASE nova;
MariaDB [(none)]> CREATE DATABASE nova_cell0;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'Welcome123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'Welcome123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'Welcome123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'Welcome123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'Welcome123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
  IDENTIFIED BY 'Welcome123';
</pre>
<li>Khai báo biến môi trường</li>
<pre>$ . admin-openrc</pre>
<li>Tạo user </li>
<pre> $ openstack user create --domain default --password-prompt nova </pre>
<li>Add the admin role to the nova user: </li>
<pre>$ openstack role add --project service --user nova admin</pre>
<li>Tạo service </li>
<pre>$ openstack service create --name nova \
  --description "OpenStack Compute" compute</pre>
  </li>Tạo endpoint</li>
  <pre>
  $ openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1
  $ openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1
  $ openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1
  </pre>
  <li>Create a Placement service </li>
  <pre>$ openstack user create --domain default --password-prompt placement</pre>
  <li>Add the Placement user to the service project with the admin role:</li>
<pre>$ openstack role add --project service --user placement admin  </pre>
  <li>Create the Placement API entry in the service catalog:   </li>
  <pre>$ openstack service create --name placement --description "Placement API" placement   </pre>
    <li>Create the Placement API service endpoints:   </li>
  <pre>
  $ openstack endpoint create --region RegionOne placement public http://controller:8778
  $ openstack endpoint create --region RegionOne placement internal http://controller:8778
  $ openstack endpoint create --region RegionOne placement admin http://controller:8778
</pre>
    <li>Cài đặt các gói cho nova và cấu hình   </li>
  <pre># apt install nova-api nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler nova-placement-api   </pre>
    <li>Sửa file /etc/nova/nova.conf   </li>
  <pre>[api_database]
connection = mysql+pymysql://nova:Welcome123@controller/nova_api
[database]
connection = mysql+pymysql://nova:Welcome123@controller/nova 
[DEFAULT]
transport_url = rabbit://openstack:Welcome123@controller
[api]
auth_strategy = keystone
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
[DEFAULT]
my_ip = 192.268.239.162
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip
[glance]
api_servers = http://controller:9292
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
# ...
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = Welcome123
</pre>
Comment dòng log_dir trong [DEFAULT] section.
    <li>Populate the nova-api database:   </li>
  <pre># su -s /bin/sh -c "nova-manage api_db sync" nova   </pre>
    <li>Register the cell0 database:   </li>
  <pre># su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
   </pre>
    <li>Create the cell1 cell:   </li>
  <pre># su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova   </pre>
    <li>Populate the nova database:   </li>
  <pre># su -s /bin/sh -c "nova-manage db sync" nova   </pre>
    <li># nova-manage cell_v2 list_cells   </li>
  <pre>+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
|  Name |                 UUID                 |           Transport URL            |               Database Connection               |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
| cell0 | 00000000-0000-0000-0000-000000000000 |               none:/               | mysql+pymysql://nova:****@controller/nova_cell0 |
| cell1 | bca97ce7-5143-41a9-93d3-75e410dd509a | rabbit://openstack:****@controller |    mysql+pymysql://nova:****@controller/nova    |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
   </pre>
     <li>Kết thúc bước cài đặt và cấu hình nova   </li>
  <pre># service nova-api restart
# service nova-consoleauth restart
# service nova-scheduler restart
# service nova-conductor restart
# service nova-novncproxy restart   </pre>
    <li>Kiểm tra xem các service của nova hoạt động tốt hay chưa bằng lệnh dưới

   </li>
  <pre> openstack compute service list
  root@controller:~# openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-scheduler   | controller | internal | enabled | up    | 2018-05-09T05:58:07.000000 |
|  5 | nova-consoleauth | controller | internal | enabled | up    | 2018-05-09T05:58:09.000000 |
|  6 | nova-conductor   | controller | internal | enabled | up    | 2018-05-09T05:58:02.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
</pre>
   
<h4>Cài đặt NEUTRON </h4>

Cài đặt và cấu hình neutron  
Tạo database cho neutron   
  <pre>mysql -u root -pWelcome123   </pre>
  <pre>MariaDB [(none)] CREATE DATABASE neutron;
  MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY 'Welcome123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY 'Welcome123';
  </pre>
Khai báo biến môi trường 
  <pre>$ . admin-openrc
   </pre>
Tạo user Neutron
  <pre>$ openstack user create --domain default --password-prompt neutron
   </pre>
Add the admin role to the neutron user:   
  <pre>$ openstack role add --project service --user neutron admin
   </pre>
Create the neutron service entity:
  <pre>$ openstack service create --name neutron \
  --description "OpenStack Networking" network   </pre>
Create the Networking service API endpoints:
  <pre>$ openstack endpoint create --region RegionOne \
  network public http://controller:9696
  
$ openstack endpoint create --region RegionOne \
  network internal http://controller:9696
 $ openstack endpoint create --region RegionOne \
  network admin http://controller:9696
 </pre>
<li>Sửa file /etc/neutron/metadata_agent.ini   </li>
  <pre>[DEFAULT]
# ...
nova_metadata_host = controller
metadata_proxy_shared_secret = Welcome123   </pre>
<li>Sửa file /etc/nova/nova.conf    </li>
  <pre>[neutron]
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = Welcome123
service_metadata_proxy = true
metadata_proxy_shared_secret = Welcome123   </pre>
<li>Cài đặt packages Neutron   </li>
 <pre> # apt install neutron-server neutron-plugin-ml2 \
  neutron-linuxbridge-agent neutron-dhcp-agent \
  neutron-metadata-agent  </pre>
  <li>Sửa file /etc/neutron/neutron.conf    </li>
  <pre>[database]
connection = mysql+pymysql://neutron:Welcome123@controller/neutron
[DEFAULT]
core_plugin = ml2
service_plugins =
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
transport_url = rabbit://openstack:Welcome123@controller


[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123


[nova]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = Welcome123
</pre>
<li>Sửa file /etc/neutron/plugins/ml2/ml2_conf.ini   </li>
<pre>[ml2]
type_drivers = flat,vlan
tenant_network_types =
mechanism_drivers = linuxbridge
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[securitygroup]
enable_ipset = true
</pre>
<li>Sửa file  /etc/neutron/plugins/ml2/linuxbridge_agent.ini   </li>
<pre>[linux_bridge]
physical_interface_mappings = provider:ens33

[vxlan]
enable_vxlan = false

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
</pre>
<li>Sửa file /etc/neutron/dhcp_agent.ini   </li>
<pre>[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true   </pre>
<li>Đồng bộ database cho neutron   </li>
<pre># su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron   </pre>
<li>Restart the Compute API service:   </li>
<pre># service nova-api restart   </pre>
<li>Khởi động lại các dịch vụ của neutron   </li>
<pre># service neutron-server restart
# service neutron-linuxbridge-agent restart
# service neutron-dhcp-agent restart
# service neutron-metadata-agent restart</pre>
<h3>6. Cài đặt trên node compute </h3>
<li>Sửa file /etc/hosts với nội dung như bên dưới </li>
<pre> 127.0.0.1       localhost compute1
 192.168.239.163     compute1
 192.168.239.162     controller   </pre>
<li>Sửa file /etc/hostname   </li>
<pre>compute1   </pre>
<li>Dùng vi mở file /etc/network/interfaces và thiết lập như dưới   </li>
<pre>
# NIC EXT
 auto ens33
 iface ens33 inet static
 address 192.168.239.163
 netmask 255.255.255.0
 gateway 192.168.239.2
 dns-nameservers 8.8.8.8   </pre>
<li>Khởi động lại network và đăng nhập lại với quyền root   </li>
<pre> ifdown -a && ifup -a   </pre>
<li>Cài đặt packages  </li>
<pre># apt install python-openstackclient </pre>
<h4>Cài đặt Nova compute </h4>
<pre># apt install nova-compute</pre>
<li>Sửa file /etc/nova/nova.conf</li>
<pre>[DEFAULT]
my_ip = 192.168.239.163
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:Welcome123@controller   

[api]
auth_strategy = keystone

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS

[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://192.168.239.162:6080/vnc_auto.html

[glance]
api_servers = http://controller:9292
[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = Welcome123
</pre>
<li>Xóa dòng log_dir trong [DEFAULT] section.   </li>

<li>Sửa file /etc/nova/nova-compute.conf   </li>
<pre>[libvirt]
virt_type = qemu   </pre>
<li>Restart the Compute service:   </li>
<pre># service nova-compute restart   </pre>
<li>Trên controller node   </li>
<pre>. admin-openrc
$ openstack compute service list --service nova-compute
+----+--------------+----------+------+---------+-------+----------------------------+
| ID | Binary       | Host     | Zone | Status  | State | Updated At                 |
+----+--------------+----------+------+---------+-------+----------------------------+
|  8 | nova-compute | compute1 | nova | enabled | up    | 2018-05-09T01:36:10.000000 |
+----+--------------+----------+------+---------+-------+----------------------------+
# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
</pre>
<li>Verify</li>
<pre>$ . admin-openrc   
$ openstack compute service list
root@controller:~# openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  5 | nova-consoleauth | controller | internal | enabled | up    | 2018-05-09T01:31:18.000000 |
|  6 | nova-scheduler   | controller | internal | enabled | up    | 2018-05-09T01:31:18.000000 |
|  7 | nova-conductor   | controller | internal | enabled | up    | 2018-05-09T01:31:21.000000 |
|  8 | nova-compute     | compute1   | nova     | enabled | down  | 2018-05-08T13:46:23.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
</pre>
<pre>
root@controller:~# openstack catalog list
+-----------+-----------+-----------------------------------------+
| Name      | Type      | Endpoints                               |
+-----------+-----------+-----------------------------------------+
| keystone  | identity  | RegionOne                               |
|           |           |   admin: http://controller:5000/v3/     |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:5000/v3/  |
|           |           | RegionOne                               |
|           |           |   public: http://controller:5000/v3/    |
|           |           |                                         |
| placement | placement | RegionOne                               |
|           |           |   admin: http://controller:8778         |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8778        |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8778      |
|           |           |                                         |
| glance    | image     | RegionOne                               |
|           |           |   admin: http://controller:9292         |
|           |           | RegionOne                               |
|           |           |   public: http://controller:9292        |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9292      |
|           |           |                                         |
| nova      | compute   | RegionOne                               |
|           |           |   public: http://controller:8774/v2.1   |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8774/v2.1    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8774/v2.1 |
|           |           |                                         |
| neutron   | network   | RegionOne                               |
|           |           |   admin: http://controller:9696         |
|           |           | RegionOne                               |
|           |           |   public: http://controller:9696        |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9696      |
|           |           |                                         |
+-----------+-----------+-----------------------------------------+
root@controller:~#
</pre>
<pre>root@controller:~# openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 4974052f-5fec-4bad-b3f0-492e175a67f4 | cirros | active |
+--------------------------------------+--------+--------+
root@controller:~#</pre>

<pre>
root@controller:~# nova-status upgrade check
+---------------------------+
| Upgrade Check Results     |
+---------------------------+
| Check: Cells v2           |
| Result: Success           |
| Details: None             |
+---------------------------+
| Check: Placement API      |
| Result: Success           |
| Details: None             |
+---------------------------+
| Check: Resource Providers |
| Result: Success           |
| Details: None             |
+---------------------------+
root@controller:~#</pre>
<h6> Cài đặt Neutron </h4>
<li>Cài đặt các thành phần:</li>
<pre># apt install neutron-linuxbridge-agent</pre>
<li> Dùng lệnh vi sửa file /etc/neutron/neutron.conf với nội dung sau:  </li>
<pre>
[DEFAULT]
transport_url = rabbit://openstack:Welcome123@controller
auth_strategy = keystone
[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123
</pre>
<li> Sửa file /etc/nova/nova.conf </li>
<pre>[neutron]
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = Welcome123</pre>
<li>Dùng lệnh vi sửa file /etc/neutron/plugins/ml2/linuxbridge_agent.ini  với nội dung:   </li>
<pre>[linux_bridge]
physical_interface_mappings = provider:ens33

[vxlan]
enable_vxlan = false

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
</pre>
<li>Khởi động lại dịch vụ   </li>
<pre> # service nova-compute restart
# service neutron-linuxbridge-agent restart</pre>
 <h4> Cài đặt HORIZON (dashboad) </h4>
<li>Cài đặt packages   </li>
<pre># apt install openstack-dashboard</pre>
<li>Chỉnh sửa file /etc/openstack-dashboard/local_settings.py với nội dung sau:   </li>
<pre>OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*']
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': '192.168.239.162:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
OPENSTACK_NEUTRON_NETWORK = {
    ...
    'enable_router': False,
    'enable_quotas': False,
    'enable_ipv6': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': False,
    'enable_firewall': False,
    'enable_vpn': False,
    'enable_fip_topology_check': False,
}
TIME_ZONE = "Asia/Ho_Chi_Minh"
(Chạy lệnh timedatectl set-timezone Asia/Ho_Chi_Minh để đồng bộ thời gian nếu cần).
</pre>
<li> Dùng vi sửa file  /etc/apache2/conf-available/openstack-dashboard.conf với nội dung sau:   </li>
<pre>WSGIApplicationGroup %{GLOBAL}   </pre>
<li>Khởi động lại dịch vụ apache2  </li>
<pre># service apache2 reload   </pre>

<h3>Sử dụng Dashboard </h3>

<li>Giao diện màn hình đăng nhập.</li>
<img src="https://github.com/anhict/images/blob/master/28.PNG?raw=true">
<li> Di chuyển đến tab Admin >> Network >> Networks >> Create Network</li>
<img src="https://github.com/anhict/images/blob/master/29.PNG?raw=true">
<img src="https://github.com/anhict/images/blob/master/30.PNG?raw=true">
<img src="https://github.com/anhict/images/blob/master/31.PNG?raw=true">

<li> Di chuyển đến tab Project >> Compute >>  Instances >> Launch Instance  để tạo các VM   </li>
<img src="https://github.com/anhict/images/blob/master/32.PNG?raw=true">
<img src="https://github.com/anhict/images/blob/master/33.PNG?raw=true">
<img src="https://github.com/anhict/images/blob/master/34.PNG?raw=true">
<img src="https://github.com/anhict/images/blob/master/35.PNG?raw=true">


<li> Quá trình tạo máy ảo : </li>
<img src="https://github.com/anhict/images/blob/master/36.PNG?raw=true">
<img src="https://github.com/anhict/images/blob/master/37.PNG?raw=true">
<li> Ping kiểm tra: </li>
<img src="https://github.com/anhict/images/blob/master/38.PNG?raw=true">







