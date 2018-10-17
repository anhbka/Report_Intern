<h3>Cài đặt Cinder ,tạo volume và launch instane bằng volume.</h3>
<h4>1. Cài đặt Cinder trên một node riêng.</h4>
<li>Lưu ý : Trên mô hình phải được cài đặt sẵn OpenStack , nếu chưa có tham khảo tại <a href="https://github.com/congto/OpenStack-Mitaka-Scripts/tree/master/OPS-Mitaka-LB-Ubuntu">đây</a></li>
<h3>1.1. Mô hình.</h3>
<img src="https://github.com/anhict/images/blob/master/Screenshot_52.png">
<h4>Trên Node Cinder thiết lập điah chỉ IP và hostname</h4>
<pre>vi /etc/network/interfaces</pre>
<pre>192.168.239.190 controller
192.168.239.191 compute1
192.168.239.192 compute2
192.168.239.193 compute3
192.168.239.194 cinder</pre>
<ul>
<li>Lưu lại file cấu hình và tiến hành yêu cầu phát phát lại địa chỉ IP :</li>
</ul>
<pre>ifdown -a <span class="pl-k">&amp;&amp;</span> ifup -a</pre>
<ul>
<li>Thiết lập file hosts :</li>
</ul>
<code>vi /etc/hosts</code>
<pre>127.0.0.1       localhost
127.0.1.1       controller
192.168.239.190 controller
192.168.239.191 compute1
192.168.239.192 compute2
192.168.239.193 compute3
192.168.239.194 cinder</pre>

<p>Chỉnh sửa tương tự trên node Controller và Compute1</p>
<ul>
<li>Reboot lại máy chủ để lấy cấu hình mới :</li>
</ul>
<h4>1.2. Cài đặt.</h4>
<ul>
<li>Ở đây mình chỉ tiến hành cài node Cinder, mặc định là OpenStack đã có và chúng ta chỉ cài thêm dịch vụ lưu trữ Block Storage (Cinder) , nếu chưa có OpenStack có thể xem tại <a href="https://github.com/congto/OpenStack-Mitaka-Scripts/tree/master/OPS-Mitaka-LB-Ubuntu/scripts">đây</a> để đồng bộ với mô hình cài node cinder .</li>
</ul>
<h4>Trên Node Controller :</h4>
<p>Cài đặt theo docs trên trang chủ https://docs.openstack.org/cinder/queens/install/cinder-controller-install-ubuntu.html </p>
<ul>
<li>Tạo cơ sở dữ liệu cho Cinder :</li>
</ul>
<p>Truy cập vào cơ sở dữ liệu và tạo databases:</p>
<pre>mysql -u root -pWelcome123</pre>

<p>Tạo cinder database:</p>

<pre>CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'Welcome123';</pre>
  <p>Chạy source admin-openrc để có quyền truy cập vào các lệnh CLI của quản trị viên :</p>
  <pre>. admin-openrc</pre>
  <p>Tạo cinder user :</p>
  <pre>openstack user create --domain default --password-prompt cinder</pre>
  
  <p>Thêm role admin vào cinder user :</p>
  <pre>openstack role add --project service --user cinder admin</pre>
  <p>Tạo các entities cinderv2 và cinderv3 :</p>
  
  <pre>openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | eb9fd245bdbc414695952e93f29fe3ac |
| name        | cinderv2                         |
| type        | volumev2                         |
+-------------+----------------------------------+

openstack service create --name cinderv3 \
  --description "OpenStack Block Storage" volumev3

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | ab3bbbef780845a1a283490d281e7fda |
| name        | cinderv3                         |
| type        | volumev3                         |
+-------------+----------------------------------+</pre>
  <p>Tạo các enpoint cho Cinder :</p>
  <pre>$ openstack endpoint create --region RegionOne \
  volumev2 public http://controller:8776/v2/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 513e73819e14460fb904163f41ef3759         |
| interface    | public                                   |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | eb9fd245bdbc414695952e93f29fe3ac         |
| service_name | cinderv2                                 |
| service_type | volumev2                                 |
| url          | http://controller:8776/v2/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
  volumev2 internal http://controller:8776/v2/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 6436a8a23d014cfdb69c586eff146a32         |
| interface    | internal                                 |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | eb9fd245bdbc414695952e93f29fe3ac         |
| service_name | cinderv2                                 |
| service_type | volumev2                                 |
| url          | http://controller:8776/v2/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
  volumev2 admin http://controller:8776/v2/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | e652cf84dd334f359ae9b045a2c91d96         |
| interface    | admin                                    |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | eb9fd245bdbc414695952e93f29fe3ac         |
| service_name | cinderv2                                 |
| service_type | volumev2                                 |
| url          | http://controller:8776/v2/%(project_id)s |
+--------------+------------------------------------------+
  $ openstack endpoint create --region RegionOne \
  volumev3 public http://controller:8776/v3/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 03fa2c90153546c295bf30ca86b1344b         |
| interface    | public                                   |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | ab3bbbef780845a1a283490d281e7fda         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
  volumev3 internal http://controller:8776/v3/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 94f684395d1b41068c70e4ecb11364b2         |
| interface    | internal                                 |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | ab3bbbef780845a1a283490d281e7fda         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+

$ openstack endpoint create --region RegionOne \
  volumev3 admin http://controller:8776/v3/%\(project_id\)s

+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 4511c28a0f9840c78bacb25f10f62c98         |
| interface    | admin                                    |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | ab3bbbef780845a1a283490d281e7fda         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+</pre>
<p>Cài đặt và cấu hình các thành phần :</p>
<p>Cài đặt các gói dịch vụ :</p>
<pre># apt install cinder-api cinder-scheduler</pre>
<p>Sửa file /etc/cinder/cinder.conf như sau :</p>
<pre>[database]
connection = mysql+pymysql://cinder:Welcome123@controller/cinder

[DEFAULT]
transport_url = rabbit://openstack:Welcome123@controller
auth_strategy = keystone
my_ip = 10.0.0.11 //(IP node Controller)


[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = Welcome123
  
  
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp</pre>
  
  
<p>Đồng bộ dữ liệu Block Storage :</p>
<pre># su -s /bin/sh -c "cinder-manage db sync" cinder</pre>
<h4>Trên node compute :</h4>
<p>Sửa file cấu hình /etc/nova/nova.conf như sau :</p>
<pre>[cinder]
os_region_name = RegionOne</pre>
<p>Khởi động lại Compute API service :</p>
<div class="highlight highlight-source-shell"><pre>service nova-api restart</pre></div>
<p>Khởi động lại dịch vụ Block Storage  :</p>
<pre>service cinder-scheduler restart
service cinder-api restart</pre>
<h4>Trên Node Cinder .</h4>
<ul>
<li>Thực hiện thêm repo OpenStack :</li>
</ul>
<div class="highlight highlight-source-shell"><pre> # apt install software-properties-common
# add-apt-repository cloud-archive:queens</pre></div>
<ul>
<li>Cập nhật lại các gói phần mềm :</li>
</ul>
<div class="highlight highlight-source-shell"><pre> apt-get -y update <span class="pl-k">&amp;&amp;</span> apt-get -y dist-upgrade</pre></div>
<ul>
<li>Cài đặt LVM</li>
</ul>
<pre># apt install lvm2 thin-provisioning-tools</pre>
<li>Tạo Physical volume :</li>
<div class="highlight highlight-source-shell"><pre><span class="pl-c"><span class="pl-c">#</span> pvcreate /dev/sdb</span>
Physical volume <span class="pl-s"><span class="pl-pds">"</span>/dev/sdb<span class="pl-pds">"</span></span> successfully created</pre></div>
<li>Tạo volume group <code>cinder-volumes</code></li>
<pre><span class="pl-c"><span class="pl-c">#</span> vgcreate cinder-volumes /dev/sdb</span>
Volume group <span class="pl-s"><span class="pl-pds">"</span>cinder-volumes<span class="pl-pds">"</span></span> successfully created</pre>
<ul>
<li>
<p>Theo mặc định công cụ quét của LVM sẽ quét toàn bộ thư mục /dev cho các thiết bị lưu trữ khối, dó đó chúng ta cần cấu hình lại
cấu hình mặc định này để LVm chỉ quét những ở thư mục mà chúng ta cấu hình và cho phép tạo volume trên đó :</p>
</li>
<li>
<p>Mở file cấu hình <code>/etc/lvm/lvm.conf</code> :</p>
</li>
</ul>
<li>
<p>Mở file cấu hình <code>/etc/lvm/lvm.conf</code> :</p>
</li>
<div class="highlight highlight-source-shell"><pre>filter = [ <span class="pl-s"><span class="pl-pds">"</span>a/sdb/<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>r/.*/<span class="pl-pds">"</span></span>]</pre></div>
<ul>
<li>cài đặt và cấu hình các dịch vụ thành phần :</li>
</ul>
<pre>apt install cinder-volume</pre>
<ul>
<li>Mở file <code>/etc/cinder/cinder.conf</code> và sửa lại như sau :</li>
</ul>
<pre>
[database]
connection = mysql+pymysql://cinder:Welcome123@controller/cinder
[DEFAULT]

transport_url = rabbit://openstack:Welcome123@controller
auth_strategy = keystone
my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS  // IP node Cinder
enabled_backends = lvm
glance_api_servers = http://controller:9292


[oslo_concurrency]
lock_path = /var/lib/cinder/tmp

[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = tgtadm


[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = Welcome123</pre>
<p>Cuối cùng chúng ta restart lại dịch vụ Block Storage bao gồm :</p>
<pre>service tgt restart
service cinder-volume restart</pre>
<h3>2. Tạo volume và launch instane :</h3>
<p>Kiểm tra các dịch vụ để kết nối với nhau thành công hay chưa :</p>

<pre>root@controller:~# openstack volume service list
+------------------+-------------+------+---------+-------+----------------------------+
| Binary           | Host        | Zone | Status  | State | Updated At                 |
+------------------+-------------+------+---------+-------+----------------------------+
| cinder-scheduler | controller  | nova | enabled | up    | 2018-07-15T13:28:04.000000 |
| cinder-volume    | cinder@lvm  | nova | enabled | up    | 2018-07-15T09:44:34.000000 |
| cinder-volume    | cinder1@lvm | nova | enabled | up    | 2018-07-15T13:27:37.000000 |
| cinder-scheduler | cinder1     | nova | enabled | up    | 2018-07-15T13:27:52.000000 |
+------------------+-------------+------+---------+-------+----------------------------+</pre>
<ul>
<li>Đăng nhập vào OpenStack từ Dashboard :</li>
</ul>

<img src="https://github.com/anhict/images/blob/master/Screenshot_51.png">
<p>Chọn Project => Compute => volumes</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_43.png">
<p>Chọn Create Volume :</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_44.png">
<p>Nếu tạo volume thành công chúng ta sẽ nhận được trạng thái như sau :</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_45.png">
<p>Chọn Launch Instane để tạo một VM mới :</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_47.png">
<img src="https://github.com/anhict/images/blob/master/Screenshot_48.png">
<img src="https://github.com/anhict/images/blob/master/Screenshot_49.png">
<img src="https://github.com/anhict/images/blob/master/Screenshot_50.png">

Tài liệu tham khảo :
1. https://github.com/hocchudong/ghichep-OpenStack/blob/master/05-Cinder/docs/cinder-install.md
2. https://docs.openstack.org/cinder/queens/install/cinder-controller-install-ubuntu.html
3. https://docs.openstack.org/cinder/queens/install/cinder-storage-install-ubuntu.html





















