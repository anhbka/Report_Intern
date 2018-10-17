<p>File cấu hình của Cinder nằm trong thư mục <code>/etc/cinder/cinder.conf</code></p>
Tham khảo :
<p>https://docs.openstack.org/cinder/queens/install/cinder-storage-install-ubuntu.html#install-and-configure-components</p>
<p>https://docs.openstack.org/cinder/queens/install/cinder-controller-install-ubuntu.html#install-and-configure-components</p>

<ul>
<li>
<p>Section <code>[DEFAULT]</code></p>
<p><code>transport_url = rabbit://openstack:Welcome123@controller</code></p>
<p>Cấu hình url cho các message điều khiển sử dụng rabbitmq làm bộ trung gian giao tiếp giữa các node.</p>
<p><code>auth_strategy = keystone</code></p>
<p>Cấu hình xác thực sử dụng Keystone.</p>
<p><code>my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS</code></p>
<p>Cấu hình địa chỉ IP của node để tiện theo dõi.</p>
<p><code>volume_name_template = volume-%s</code></p>
<p>Template mẫu sử dụng để tạo tên cho các volume (tên volume bắt đầu bằng <code>“volume- “</code> )</p>
<p><code>enabled_backends = lvm</code></p>
<ul>
<li>
<p>Cấu hình backend muốn sử dụng, ở đây là LVM</p>
</li>
<li>
<p>Đối với multiple backend có thể liệt kê các backend dùng, chỉ cần dấu phẩy giữa các backend (ví dụ : <code>enable_backends = lvm,nfs,glusterfs</code>)</p>
</li>
</ul>
<p><code>glance_api_servers = http://controller:9292</code></p>
<p>API cung cấp image. (sử dụng để upload image nếu muốn volume dùng để boot được máy ảo)</p>
</li>
<li>
<p>Section <code>[database]</code>:</p>
<pre><code> [database]
connection = mysql+pymysql://cinder:Welcome123@controller/cinder
</code></pre>
<p>Cấu hình kết nối với database trên <code>controller</code> node để lưu trữ và truy xuất dữ liệu</p>
</li>
<li>
<p>Section <code>[keystone_authtoken]</code></p>
<pre><code> [keystone_authtoken]
 auth_uri = http://controller:5000
 auth_url = http://controller:5000
 memcached_servers = controller:11211
 auth_type = password
 project_domain_name = default
 user_domain_name = default
 project_name = service
 username = cinder
 password = Welcome123
</code></pre>
<p>Cấu hình dịch vụ xác thực sử dụng Keystone và định danh cho service cinder.</p>
</li>
<li>
<p>Section <code>[lvm]</code></p>
<pre><code> [lvm]
 volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
 volume_group = cinder-volumes
 iscsi_protocol = iscsi
 iscsi_helper = tgtadm
</code></pre>
<p>Cấu hình backend sử dụng lvm, chia sẻ volume group mặc định là <code>cinder-volumes</code>, sử dụng giao thức iCSI để kết nối giữa compute và cinder được tạo ra với tgtadm.</p>
</li>
</ul>
<p>Cấu hình mặc định cho volume backend là sử dụng local volumes quản lý bởi LVM. Driver này hỗ trợ giao thức iCSI.</p>
<p>Cấu hình trong file <code>cinder.conf</code> như sau:</p>
<pre>[DEFAULT]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
iscsi_protocol = iscsi
lvm_type = default		# Loại lvm volume (default, thin, hoặc auto)
volume_group = cinder-volumes	# tên của volume group đã tạo cho lvm</pre>

