
<h3>1. Các lệnh cơ bản thường dùng trong Neutron</h3>

<pre>root@controller:~# openstack network list
+--------------------------------------+----------+--------------------------------------+
| ID                                   | Name     | Subnets                              |
+--------------------------------------+----------+--------------------------------------+
| 594a203a-b97a-4fdc-9e65-13859be9abd0 | provider | ac039666-3a6a-4cfb-8f33-7ff83f0156bd |
+--------------------------------------+----------+--------------------------------------+</pre>

<pre>root@controller:~# openstack subnet list
+--------------------------------------+---------------+--------------------------------------+------------------+
| ID                                   | Name          | Network                              | Subnet           |
+--------------------------------------+---------------+--------------------------------------+------------------+
| ac039666-3a6a-4cfb-8f33-7ff83f0156bd | sub1-provider | 594a203a-b97a-4fdc-9e65-13859be9abd0 | 192.168.239.0/24 |
+--------------------------------------+---------------+--------------------------------------+------------------+</pre>

<pre>root@controller:~# openstack security group list
+--------------------------------------+---------+------------------------+----------------------------------+
| ID                                   | Name    | Description            | Project                          |
+--------------------------------------+---------+------------------------+----------------------------------+
| 672039e7-960e-47e1-999c-abc724b2fa70 | default | Default security group | bdf777efa371433ca3bc6f58d51f11d0 |
| a14a11a8-441d-4ff5-9020-a6b373b981f2 | default | Default security group | 7ad160531200440fbcc2e241a20faaee |
+--------------------------------------+---------+------------------------+----------------------------------+</pre>
<pre>root@controller:~# openstack flavor list
+----+---------+-----+------+-----------+-------+-----------+
| ID | Name    | RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+---------+-----+------+-----------+-------+-----------+
| 0  | m1.nano |  64 |    1 |         0 |     1 | True      |
+----+---------+-----+------+-----------+-------+-----------+</pre>
<p>Show thông tin một network</p>
<p>Để xem thông tin chi tiết của network, sử dụng câu lệnh neutron net-show hoặc openstack network show</p>
<pre>root@controller:~# neutron net-show provider
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| availability_zone_hints   |                                      |
| availability_zones        | nova                                 |
| created_at                | 2018-06-01T07:41:52Z                 |
| description               |                                      |
| id                        | 594a203a-b97a-4fdc-9e65-13859be9abd0 |
| ipv4_address_scope        |                                      |
| ipv6_address_scope        |                                      |
| mtu                       | 1500                                 |
| name                      | provider                             |
| port_security_enabled     | True                                 |
| project_id                | bdf777efa371433ca3bc6f58d51f11d0     |
| provider:network_type     | flat                                 |
| provider:physical_network | provider                             |
| provider:segmentation_id  |                                      |
| revision_number           | 5                                    |
| router:external           | True                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   | ac039666-3a6a-4cfb-8f33-7ff83f0156bd |
| tags                      |                                      |
| tenant_id                 | bdf777efa371433ca3bc6f58d51f11d0     |
| updated_at                | 2018-06-01T07:41:54Z                 |
+---------------------------+--------------------------------------+</pre>

<pre>root@controller:~# neutron net-list
+--------------------------------------+----------+----------------------------------+-------------------------------------------------------+
| id                                   | name     | tenant_id                        | subnets                                               |
+--------------------------------------+----------+----------------------------------+-------------------------------------------------------+
| 594a203a-b97a-4fdc-9e65-13859be9abd0 | provider | bdf777efa371433ca3bc6f58d51f11d0 | ac039666-3a6a-4cfb-8f33-7ff83f0156bd 192.168.239.0/24 |
+--------------------------------------+----------+----------------------------------+-------------------------------------------------------+</pre>


<pre>root@controller:~# neutron agent-list
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| id                                   | agent_type         | host       | availability_zone | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| 4759c4f7-73d9-43d4-ab5b-b449766bf099 | Linux bridge agent | compute1   |                   | :-)   | True           | neutron-linuxbridge-agent |
| 4dc6ba32-7513-40f5-ba16-e332626dc46e | DHCP agent         | controller | nova              | :-)   | True           | neutron-dhcp-agent        |
| 853d2bec-26b2-4557-9524-895721ceaa12 | Linux bridge agent | controller |                   | :-)   | True           | neutron-linuxbridge-agent |
| a669c5a2-74e7-471c-b189-18c783e13760 | Metadata agent     | controller |                   | :-)   | True           | neutron-metadata-agent    |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+</pre>

<h3>2. Ghi chép file cấu hình Neutron </h3>
<p>- Chỉnh sửa file /etc/neutron/neutron.conf </p>
<p>+ Cấu hình truy cập cơ sở dữ liệu </p>
<pre[database]
connection = mysql+pymysql://neutron:Welcome123@controller/neutron</pre>
<pre>[DEFAULT]
core_plugin = ml2
service_plugins =</pre>
<p>+ Cấu hình plugin mà Neutron sẽ sử dụng. </p>

<pre>transport_url = rabbit://openstack:RABBIT_PASS@controller</pre>
<p>+ Cấu hình kết nối  RabbitMQ </p>
<pre>auth_strategy = keystone</pre>
<p>+ Loại hình xác thực sử dụng Keystone</p>
<pre>[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = Welcome123</pre>
<p>+ Cấu hình truy cập dịch vụ Identity và cấu hình Networking để thông báo cho Compute về các thay đổi cấu trúc liên kết mạng:</p>
<pre>[DEFAULT]
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[nova]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = NOVA_PASS</pre>
<p>+ Cấu hình ML2 plugin </p>
<pre>type_drivers = flat,vlan</pre>
<p>+ Loại driver được sử dụng </p>
<p>* Flat : Tất cả các instances nằm trong cùng một mạng, và có thể chia sẻ với hosts. Không hề sử dụng VLAN tagging hay hình thức tách biệt về network khác.</p>
<pre>tenant_network_types =</pre>
<p>+ Vô hiệu hóa self-service networks </p>
<pre>mechanism_drivers = linuxbridge</pre>
<p>+ Cơ chế network sử dụng linuxbridge </p>
<pre>extension_drivers = port_security</pre>
<p>Kích hoạt trình điều khiển mở rộng bảo mật </p>
<pre>flat_networks = provider</pre>
<p>Cấu hình network sử dụng provider</pre>
<pre>enable_ipset = true</pre>
<p>Mở ipset để tăng hiệu quả của các quy tắc nhóm bảo mật</p>
<pre> Cấu hình file /etc/neutron/plugins/ml2/linuxbridge_agent.ini </pre>
<pre>[vxlan]
enable_vxlan = false</pre>
<p> Vô hiệu hóa vxlan </p>
<pre>[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver</pre>
<p>Bật security groups và cấu hình Linux brigde iptables firewall drivers </p>
<pre>[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true</pre>
<p>cấu hình trình điều khiển giao diện brigde Linux, trình điều khiển DHCP Dnsmasq và kích hoạt metadata được phân lập để các trường hợp trên mạng của nhà cung cấp có thể truy cập metadata qua mạng</p>
<pre>[neutron]
# ...
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
metadata_proxy_shared_secret = Welcome123</pre>
<p>+ Cấu hình tham số truy cập và và mở metadata proxy </p>

<pre>su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron</pre>
<p>+ Đồng bộ dữ liệu </p>
<pre>service nova-api restart </pre>
<p>Restart lại nova-api </p>
<pre># service neutron-server restart
# service neutron-linuxbridge-agent restart
# service neutron-dhcp-agent restart
# service neutron-metadata-agent restart</pre>
<p>Restart the Networking services.</p>

