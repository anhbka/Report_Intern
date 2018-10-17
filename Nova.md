<h3>1. Quản lý flavor </h3>
<p>Instance flavor là template của máy ảo và nó chỉ ra máy ảo thuộc loại nào. Ngay sau khi cài đặt OpenStack cloud, người dùng sẽ có trước một vài các flavor. Bạn có thể thêm hoặc xóa các flavor có sẵn.</p>
<table>
<thead>
<tr>
<th>Element</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>Name</td>
<td>Tên mô tả</td>
</tr>
<tr>
<td>Memory MB</td>
<td>RAM máy ảo (megabytes)</td>
</tr>
<tr>
<td>Disk</td>
<td>Ổ đĩa máy ảo (gigabytes). Đây là ephemeral disk mà base image được copy lên. Khi boot từ volume thì nó không được sử dụng</td>
</tr>
<tr>
<td>Ephemeral</td>
<td>Kích thước của ephemeral data disk số 2. Đây là đĩa trống, chưa được format và chỉ tồn tại khi máy ảo chạy</td>
</tr>
<tr>
<td>Swap</td>
<td>Đây là tùy chọn cho swap của máy ảo. Giá trị mặc định là 0</td>
</tr>
<tr>
<td>VCPUs</td>
<td>Số lượng CPUs ảo của máy ảo</td>
</tr>
<tr>
<td>Is Public</td>
<td>Quy định flavor có thể được dùng bởi tất cả các user hay chỉ những user trong project nào đó</td>
</tr>
<tr>
<td>Extra Specs</td>
<td>Quy định flavor được dùng trên node compute nào</td>
</tr></tbody></table>
<p>Tạo 1 flavor </p>
<pre>
root@controller:~# openstack flavor create --id 1 --vcpus 1 --ram 64 --disk 1 m1.nano1
+----------------------------+----------+
| Field                      | Value    |
+----------------------------+----------+
| OS-FLV-DISABLED:disabled   | False    |
| OS-FLV-EXT-DATA:ephemeral  | 0        |
| disk                       | 1        |
| id                         | 1        |
| name                       | m1.nano1 |
| os-flavor-access:is_public | True     |
| properties                 |          |
| ram                        | 64       |
| rxtx_factor                | 1.0      |
| swap                       |          |
| vcpus                      | 1        |
+----------------------------+----------+
root@controller:~#</pre>
<p>- Hiển thị danh sách các flavor </p>
<pre>root@controller:~# openstack flavor list
+----+---------+-----+------+-----------+-------+-----------+
| ID | Name    | RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+---------+-----+------+-----------+-------+-----------+
| 0  | m1.nano |  64 |    1 |         0 |     1 | True      |
+----+---------+-----+------+-----------+-------+-----------+
root@controller:~#</pre>
<p>- Hiển thị thông tin chi tiết 1 flavor</p>
<pre>root@controller:~# openstack flavor show m1.nano
+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| access_project_ids         | None    |
| disk                       | 1       |
| id                         | 0       |
| name                       | m1.nano |
| os-flavor-access:is_public | True    |
| properties                 |         |
| ram                        | 64      |
| rxtx_factor                | 1.0     |
| swap                       |         |
| vcpus                      | 1       |
+----------------------------+---------+
root@controller:~#</pre>
<p>Tạo mới flavor với tên m10.tiny có 5GB disk, 400MB RAM và 1 vCPU, sử dụng câu lệnh </p>
<pre>root@controller:~# nova flavor-create --is-public true m10.tiny auto 400 5 1
+--------------------------------------+----------+-----------+------+-----------+------+-------+-------------+-----------+
| ID                                   | Name     | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
+--------------------------------------+----------+-----------+------+-----------+------+-------+-------------+-----------+
| 098c0f63-9626-4879-b654-566ce22f099a | m10.tiny | 400       | 5    | 0         |      | 1     | 1.0         | True      |
+--------------------------------------+----------+-----------+------+-----------+------+-------+-------------+-----------+
root@controller:~#</pre>
<p>Xóa flavor, sử dụng câu lệnh nova flavor-delete </p>
<h3>2. Quản lí và truy cập máy ảo sử dụng keypair</h3>
<p>SSH cho phép bạn xác thực user bằng việc sử dụng private-public keypair. Máy ảo chạy với public key chỉ có thể được sử dụng bởi người nắm giữ private key.</p>

<p>OpenStack có thể lưu public key và đưa nó vào bên trong instance ở thời điểm máy ảo được khởi chạy. Nhiệm vụ của bạn là phải giữ private key ở trạng thái bảo mật. Nếu bạn mấy key, bạn sẽ không thể lấy lại. Trong trường hợp đó bạn nên bỏ public key ở máy ảo và generate ra một cặp keys khác. Nếu ai đó có private key, họ sẽ có thể truy cập được vào máy ảo của bạn.</p>
<p>Để tạo keypair, chạy câu lệnh sau:</p>
<code>nova keypair-add apresskey1 > ~/apresskey</code>
<pre>root@controller:~# cat ~/apresskey
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAosSPelcmK9lWn4TW/pClWozYAhZGlP+stPExgssu+yVgrJ1X
43X/+tUF08+6kjSaW1dqYlstvTBubaZP1QGZaXbjEi8yeoxZG/AIHmeE1Wj8jZZw
6fvlJtWzMCPdrdHpGoblUXSQB3b4xC1aiF6wMfp+nB2q2DRF6KcxfZijxU54B2UL
NzQ9nMm8LOOsGhLin197JPyW1bMFnpiqvHPi1wmqDT78M4jq7WNu9eCfaaB//I6R
vSSd0r3B1I0n2lwvcqT+hDgwFKi3N5k1zhhrx+4geWgVOpi7UwC0IB3gECzYFDoY
aGCoJEhfJRdjfD85gbEE0vCr0B2po1oww4mv4wIDAQABAoIBADBbs8EGSWn3rXvB
TNrfALGRbM/Z7GhyOc6cZjhUw4WMSleee7ExqrbMOWn/qo+rnzyKESpdqo4t6HEd
W1SOoBSsZLRPX3D3Z7YcL11RJi79fSNX0f5Cf4d1MEKaNU9iMR5Xe6QivHPrTeeD
DgW8FB8VLC6Xxd1sUmTX36VQB0AkIcrRylFyJHoIa5EcTnabJUfnpomHiScZb3Gi
GPuXb/ZKf3hqvoCduaSW+FjHSx8OHEdVALe1SPq90FT8mWkpCECEmbVzqXzs3P+w
e9YIjENQOcKTEOe2qT2TXfWNup32OSZuuTJ7r4bCmRMhLxrdIkUTmDQsw+xNqjim
vY8TG+ECgYEAzZS8bLYNy/lOKIZvH2IR6dSXG28JMBwArPpCEmgUdHVtJe4pUl8e
fLcCGSi0hMg7f6mt9sOfGbW1+88a6JDvTgmUauOitWS1mbZ6eT35SiozlZgdAIIX
p1RvkqVkGk4Yj4f0KhsSXscIOWOjPxpFHAQN2Xd6HV/M9EvB8KjnLDMCgYEAyq/T
2OCfhcq0LuD6RJq99Y6elaCZDdaTFj6vkirYvP7MyN9GllcMgz6kYFzG3htS5HUz
1HyeKLkVdZlcBbPqlK+eGJ29NmqakOvY44awiqfHwUnBJ5c05o15nZUGeWnEkSrj
rrrXYM0EndnTg0u298RfxTBnlG7mlf5mnWcovZECgYEAxJZssNBO1TTr5pjXfn07
gA1JCnTdpmHAy6jssclRdiQsYc8jOJ+4+a3Pldt09Fy9eND7iDN82wsGoWtk4exm
yosioCxaFXfeqMT0zSfUUXWVqoGxiiDdGagGoYcC+Jyho+9wLyuAH53YYXjETL2E
RMwjqkc0QQ6xYRNovAfoOD0CgYBLK97UBqrjQgSFhmcLXqCpG9XxBHj/St+OVn1j
JoTvw0hMD5LsWyiG3Iq2OnJ/GX8qv9UTL4yw6cPts40PiGSt9FwcIRR1xB/DM9Vi
vSdopUVOiH4cotW51CqQqR6XlQSUGmYK/by3aBIYQRtTDJe1WJ10Url3sZHPe2Sv
Z/0SIQKBgQCjGm131E4ON+f4seueR3txet3IqcZAqAmtkJ1dVwjEiu5shJTKaDaR
FoUrznO1JVSoqSOp2PR9PqMrpZBy81oHozDIxJPuaPT4vEa3I7L9qK5ZaN/VkCiW
RXJPMxNVif2qkNkjKxReVCpKzfGb8/D4eSbL8ckpyA5rfpOlpR+0fA==
-----END RSA PRIVATE KEY-----

root@controller:~#</pre>
<p> - Kiểm tra danh sách keypair </p>
<pre>root@controller:~# nova keypair-list
+------------+------+-------------------------------------------------+
| Name       | Type | Fingerprint                                     |
+------------+------+-------------------------------------------------+
| apresskey1 | ssh  | c1:e1:93:2f:db:94:21:44:c9:dc:b6:20:b9:21:b2:2d |
+------------+------+-------------------------------------------------+
root@controller:~# openstack keypair list
+------------+-------------------------------------------------+
| Name       | Fingerprint                                     |
+------------+-------------------------------------------------+
| apresskey1 | c1:e1:93:2f:db:94:21:44:c9:dc:b6:20:b9:21:b2:2d |
+------------+-------------------------------------------------+
root@controller:~#</pre>
<p>Trước khi SSH client có thể dùng private key, bạn cần chắc chắn đã cấp quyền truy cập cho nó:</p>
<pre>root@controller:~# chmod 600 apresskey
root@controller:~# ls
admin-openrc  apresskey  cirros-0.3.5-x86_64-disk.img  demo-openrc
root@controller:~# ls -l apresskey
-rw------- 1 root root 1680 Jun 19 09:18 apresskey
root@controller:~#</pre>
<h3>3. Khởi tạo, tắt và hủy máy ảo</h3>
<p>Để khởi tạo một máy ảo, bạn cần cung cấp 3 yếu tố: tên máy ảo, flavor, và source của máy ảo. Source ở đây có thể là image, snapshot hoặc volume. Bạn cũng có thể thêm một vài yếu tố tùy chọn như keypair, security group, user data files, và volume.</p>
<p>Dưới đây là câu lệnh để khởi rạo máy ảo:</p>
<code>nova boot --flavor FLAVOR_ID --image IMAGE_ID --key-name KEY_NAME \
 --user-data USER_DATA_FILE --security-groups SEC_GROUP_NAME --meta KEY=VALUE \ INSTANCE_NAME</code>
 <p>- Để kết nối máy ảo vừa tạo với console trên trình duyệt bằng noVNC client, chạy câu lệnh sau:</p>
 <pre>root@controller:~# nova get-vnc-console 1 novnc
+-------+--------------------------------------------------------------------------------------+
| Type  | Url                                                                                  |
+-------+--------------------------------------------------------------------------------------+
| novnc | http://192.168.239.129:6080/vnc_auto.html?token=53c90fbd-f8ad-4e95-8c21-a3b00f2ee19d |
+-------+--------------------------------------------------------------------------------------+
root@controller:~#</pre>
<img src="https://github.com/anhict/images/blob/master/Screenshot_3.png">
<p>Để xóa máy ảo, sử dụng câu lệnh nova delete</p>
<pre>root@controller:~# nova list
+--------------------------------------+------+--------+------------+-------------+-------------------------+
| ID                                   | Name | Status | Task State | Power State | Networks                |
+--------------------------------------+------+--------+------------+-------------+-------------------------+
| 0fd9584b-21ed-4e0b-97ce-377524b91b18 | 1    | ACTIVE | -          | Running     | selfservice=10.10.10.10 |
| 9561c549-5564-420d-982e-e9db0f7f0324 | vm01 | ACTIVE | -          | Running     | selfservice=10.10.10.6  |
+--------------------------------------+------+--------+------------+-------------+-------------------------+
root@controller:~# nova delete 1
Request to delete server 1 has been accepted.
root@controller:~# nova list
+--------------------------------------+------+--------+------------+-------------+------------------------+
| ID                                   | Name | Status | Task State | Power State | Networks               |
+--------------------------------------+------+--------+------------+-------------+------------------------+
| 9561c549-5564-420d-982e-e9db0f7f0324 | vm01 | ACTIVE | -          | Running     | selfservice=10.10.10.6 |
+--------------------------------------+------+--------+------------+-------------+------------------------+
root@controller:~#</pre>
<p>Để start máy ảo, sử dụng <code>nova start ID </code> </p>
<h3>4. Quản lí snapshot </h3>
<p>OpenStack có thể tạo snapshot khi máy ảo đang chạy. snapshot tương tự như image, người dùng có thể tạo mới máy ảo từ snapshot.</p>

<p>Kiểm tra xem có image hay máy ảo nào đang chạy không:</p>
<pre>root@controller:~# openstack image list
+--------------------------------------+---------+--------+
| ID                                   | Name    | Status |
+--------------------------------------+---------+--------+
| 95502eef-fadf-4f58-ad5f-bd3872822ed7 | cirros  | active |
| d1019cd9-8dbf-4a0b-a33b-070798c85dab | cirros  | active |
| 99c50513-9d32-47cf-b4bf-7eec42339621 | cirros1 | active |
+--------------------------------------+---------+--------+</pre>
<p>Tạo snapshot cho vm01 bằng câu lệnh: </p>
<pre>root@controller:~# nova image-create vm01 vm01_snap</pre>
<p>Kiểm tra </p>
<pre>root@controller:~# openstack image list
+--------------------------------------+-----------+--------+
| ID                                   | Name      | Status |
+--------------------------------------+-----------+--------+
| 95502eef-fadf-4f58-ad5f-bd3872822ed7 | cirros    | active |
| d1019cd9-8dbf-4a0b-a33b-070798c85dab | cirros    | active |
| 99c50513-9d32-47cf-b4bf-7eec42339621 | cirros1   | active |
| b9bb75e0-764d-42c8-b6fc-973780fdd977 | vm01_snap | active |
+--------------------------------------+-----------+--------+</pre>
<pre>root@controller:~# nova list
+--------------------------------------+------+--------+------------+-------------+------------------------+
| ID                                   | Name | Status | Task State | Power State | Networks               |
+--------------------------------------+------+--------+------------+-------------+------------------------+
| 9561c549-5564-420d-982e-e9db0f7f0324 | vm01 | ACTIVE | -          | Running     | selfservice=10.10.10.6 |
+--------------------------------------+------+--------+------------+-------------+------------------------+
root@controller:~# nova stop vm01
Request to stop server vm01 has been accepted.
root@controller:~#</pre>
<pre>root@controller:~# nova list
+--------------------------------------+------+---------+------------+-------------+------------------------+
| ID                                   | Name | Status  | Task State | Power State | Networks               |
+--------------------------------------+------+---------+------------+-------------+------------------------+
| 9561c549-5564-420d-982e-e9db0f7f0324 | vm01 | SHUTOFF | -          | Shutdown    | selfservice=10.10.10.6 |
+--------------------------------------+------+---------+------------+-------------+------------------------+
root@controller:~#</pre>
<p>Sử dụng lệnh <code>nova image-create </code> tạo máy snapshot </p>
<pre>root@controller:~# nova image-create --poll vm01 myInstanceSnapshot

Server snapshotting... 100% complete
Finished
root@controller:~#</pre>
<pre>root@controller:~# openstack image list
+--------------------------------------+--------------------+--------+
| ID                                   | Name               | Status |
+--------------------------------------+--------------------+--------+
| 95502eef-fadf-4f58-ad5f-bd3872822ed7 | cirros             | active |
| d1019cd9-8dbf-4a0b-a33b-070798c85dab | cirros             | active |
| 99c50513-9d32-47cf-b4bf-7eec42339621 | cirros1            | active |
| f9d68b27-3c3b-48b2-87d4-6e02af9ad9a5 | myInstanceSnapshot | active |
| b9bb75e0-764d-42c8-b6fc-973780fdd977 | vm01_snap          | active |
+--------------------------------------+--------------------+--------+
root@controller:~#</pre>
<pre>root@controller:~# glance image-download --file snapshot.raw f9d68b27-3c3b-48b2-87d4-6e02af9ad9a5</pre>
<pre>root@controller:~# nova hypervisor-list
+--------------------------------------+---------------------+-------+---------+
| ID                                   | Hypervisor hostname | State | Status  |
+--------------------------------------+---------------------+-------+---------+
| c668a2d1-20e6-4afe-889f-092e42596fa5 | compute1            | up    | enabled |
+--------------------------------------+---------------------+-------+---------+
root@controller:~#</pre>
<pre>root@controller:~# nova hypervisor-servers compute1
+--------------------------------------+-------------------+--------------------------------------+---------------------+
| ID                                   | Name              | Hypervisor ID                        | Hypervisor Hostname |
+--------------------------------------+-------------------+--------------------------------------+---------------------+
| 9561c549-5564-420d-982e-e9db0f7f0324 | instance-00000006 | c668a2d1-20e6-4afe-889f-092e42596fa5 | compute1            |
+--------------------------------------+-------------------+--------------------------------------+---------------------+
root@controller:~#</pre>

<h3>5. Quản lí quota</h3>
<p>Quota giới hạn số lượng resource. Số lượng resource mặc định được cho phép cho mỗi tenant được định nghĩa trong file config của nova (/etc/nova/nova.conf).</p>
<pre># Number of instances allowed per project (integer value)
quota_instances=10
# Number of instance cores allowed per project (integer value)
quota_cores=20
# Megabytes of instance RAM allowed per project (integer value)
quota_ram=51200
# Number of floating IPs allowed per project (integer value)
quota_floating_ips=10
# Number of fixed IPs allowed per project (this should be at least the number
# of instances allowed) (integer value)
quota_fixed_ips=-1
# Number of metadata items allowed per instance (integer value)
quota_metadata_items=128
# Number of injected files allowed (integer value)
quota_injected_files=5
# Number of bytes allowed per injected file (integer value)
quota_injected_file_content_bytes=10240
# Length of injected file path (integer value)
quota_injected_file_path_length=255
# Number of security groups per project (integer value)
quota_security_groups=10
# Number of security rules per security group (integer value)
quota_security_group_rules=20
# Number of key pairs per user (integer value)
quota_key_pairs=100</pre>

<pre>root@controller:~# nova quota-show
+-----------------------------+-------+
| Quota                       | Limit |
+-----------------------------+-------+
| instances                   | 10    |
| cores                       | 20    |
| ram                         | 51200 |
| metadata_items              | 128   |
| injected_files              | 5     |
| injected_file_content_bytes | 10240 |
| injected_file_path_bytes    | 255   |
| key_pairs                   | 100   |
| server_groups               | 10    |
| server_group_members        | 10    |
+-----------------------------+-------+
root@controller:~#</pre>
<pre>root@controller:~# nova usage-list
Usage from 2018-05-22 to 2018-06-20:
+----------------------------------+---------+--------------+-----------+---------------+
| Tenant ID                        | Servers | RAM MB-Hours | CPU Hours | Disk GB-Hours |
+----------------------------------+---------+--------------+-----------+---------------+
| 74455f45a933413f81d9c36751c9dfd0 | 2       | 43014.96     | 672.11    | 672.11        |
+----------------------------------+---------+--------------+-----------+---------------+
root@controller:~#</pre>
<pre>root@controller:~# nova show vm01
+--------------------------------------+----------------------------------------------------------+
| Property                             | Value                                                    |
+--------------------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig                    | AUTO                                                     |
| OS-EXT-AZ:availability_zone          | nova                                                     |
| OS-EXT-SRV-ATTR:host                 | compute1                                                 |
| OS-EXT-SRV-ATTR:hostname             | vm01                                                     |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | compute1                                                 |
| OS-EXT-SRV-ATTR:instance_name        | instance-00000006                                        |
| OS-EXT-SRV-ATTR:kernel_id            |                                                          |
| OS-EXT-SRV-ATTR:launch_index         | 0                                                        |
| OS-EXT-SRV-ATTR:ramdisk_id           |                                                          |
| OS-EXT-SRV-ATTR:reservation_id       | r-p5603cx8                                               |
| OS-EXT-SRV-ATTR:root_device_name     | /dev/vda                                                 |
| OS-EXT-SRV-ATTR:user_data            | -                                                        |
| OS-EXT-STS:power_state               | 4                                                        |
| OS-EXT-STS:task_state                | -                                                        |
| OS-EXT-STS:vm_state                  | stopped                                                  |
| OS-SRV-USG:launched_at               | 2018-06-19T03:14:32.000000                               |
| OS-SRV-USG:terminated_at             | -                                                        |
| accessIPv4                           |                                                          |
| accessIPv6                           |                                                          |
| config_drive                         |                                                          |
| created                              | 2018-06-19T03:13:50Z                                     |
| description                          | -                                                        |
| flavor:disk                          | 1                                                        |
| flavor:ephemeral                     | 0                                                        |
| flavor:extra_specs                   | {}                                                       |
| flavor:original_name                 | m1.nano                                                  |
| flavor:ram                           | 64                                                       |
| flavor:swap                          | 0                                                        |
| flavor:vcpus                         | 1                                                        |
| hostId                               | f16a249370d43d4c283a2f439276e8022bba7522b665dede4be144d9 |
| host_status                          | UP                                                       |
| id                                   | 9561c549-5564-420d-982e-e9db0f7f0324                     |
| image                                | cirros1 (99c50513-9d32-47cf-b4bf-7eec42339621)           |
| key_name                             | apresskey1                                               |
| locked                               | False                                                    |
| metadata                             | {}                                                       |
| name                                 | vm01                                                     |
| os-extended-volumes:volumes_attached | []                                                       |
| security_groups                      | default                                                  |
| selfservice network                  | 10.10.10.6                                               |
| status                               | SHUTOFF                                                  |
| tags                                 | []                                                       |
| tenant_id                            | 74455f45a933413f81d9c36751c9dfd0                         |
| updated                              | 2018-06-19T07:59:20Z                                     |
| user_id                              | 913cb7b99aee4f39a4465deddc3c37ca                         |
+--------------------------------------+----------------------------------------------------------+
root@controller:~#</pre>
<pre>root@controller:~# openstack service show nova
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 2967d2b73ece498dbc643cd881bb4b9c |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
root@controller:~#</pre>
<p>Bạn cũng sẽ có thể cần phải check log, bằng câu lệnh lsof, bạn có thể xem danh sách log và service đang sử dụng chúng:</p>
<pre>root@controller:~# lsof /var/log/nova/*
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
nova-sche 2352 nova    3w   REG  252,0  1868001 1838745 /var/log/nova/nova-scheduler.log
nova-api  2386 nova    3w   REG  252,0 12591175 1839151 /var/log/nova/nova-api.log
nova-cond 2397 nova    3w   REG  252,0    46558 1838705 /var/log/nova/nova-conductor.log
nova-cons 2404 nova    3w   REG  252,0    53629 1836807 /var/log/nova/nova-consoleauth.log
nova-novn 2411 nova    3w   REG  252,0    40504 1838711 /var/log/nova/nova-novncproxy.log
nova-api  2764 nova    3w   REG  252,0 12591175 1839151 /var/log/nova/nova-api.log
nova-api  2767 nova    3w   REG  252,0 12591175 1839151 /var/log/nova/nova-api.log
root@controller:~#</pre>
<pre>root@controller:~# openstack hypervisor list
+----+---------------------+-----------------+-----------------+-------+
| ID | Hypervisor Hostname | Hypervisor Type | Host IP         | State |
+----+---------------------+-----------------+-----------------+-------+
|  1 | compute1            | QEMU            | 192.168.239.130 | up    |
+----+---------------------+-----------------+-----------------+-------+
root@controller:~#</pre>
<h3>6. Quản lí volume </h3>
<p>Check xem nova servers đã được start hết hay chưa:</p>
<pre>root@controller:~# systemctl status *nova* -n 0
● nova-scheduler.service - OpenStack Compute Scheduler
   Loaded: loaded (/lib/systemd/system/nova-scheduler.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2018-06-19 13:42:42 +07; 1h 52min ago
 Main PID: 2352 (nova-scheduler)
    Tasks: 1
   Memory: 119.1M
      CPU: 8.524s
   CGroup: /system.slice/nova-scheduler.service
           └─2352 /usr/bin/python /usr/bin/nova-scheduler --config-file=/etc/nova/nova.conf --log-file=/var/log/nova/nova-scheduler.log

● nova-novncproxy.service - OpenStack Compute novncproxy
   Loaded: loaded (/lib/systemd/system/nova-novncproxy.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2018-06-19 13:42:44 +07; 1h 52min ago
 Main PID: 2411 (nova-novncproxy)
    Tasks: 1
   Memory: 116.2M
      CPU: 4.036s
   CGroup: /system.slice/nova-novncproxy.service
           └─2411 /usr/bin/python /usr/bin/nova-novncproxy --config-file=/etc/nova/nova.conf --log-file=/var/log/nova/nova-novncproxy.log

● nova-conductor.service - OpenStack Compute Conductor
   Loaded: loaded (/lib/systemd/system/nova-conductor.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2018-06-19 13:42:44 +07; 1h 52min ago
 Main PID: 2397 (nova-conductor)
    Tasks: 1
   Memory: 126.4M
      CPU: 36.930s
   CGroup: /system.slice/nova-conductor.service
           └─2397 /usr/bin/python /usr/bin/nova-conductor --config-file=/etc/nova/nova.conf --log-file=/var/log/nova/nova-conductor.log

● nova-consoleauth.service - OpenStack Compute Console
   Loaded: loaded (/lib/systemd/system/nova-consoleauth.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2018-06-19 13:42:44 +07; 1h 52min ago
 Main PID: 2404 (nova-consoleaut)
    Tasks: 1
   Memory: 115.7M
      CPU: 9.529s
   CGroup: /system.slice/nova-consoleauth.service
           └─2404 /usr/bin/python /usr/bin/nova-consoleauth --config-file=/etc/nova/nova.conf --log-file=/var/log/nova/nova-consoleauth.log

● nova-api.service - OpenStack Compute API
   Loaded: loaded (/lib/systemd/system/nova-api.service; enabled; vendor preset: enabled)</pre>

<p>Câu lệnh dùng để quản lí volume:</p>
<table>
<thead>
<tr>
<th>Command</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>server add volume</td>
<td>Gán volume cho server</td>
</tr>
<tr>
<td>volume create</td>
<td>Thêm mới volume</td>
</tr>
<tr>
<td>volume delete</td>
<td>Xóa volume</td>
</tr>
<tr>
<td>server remove volume</td>
<td>Gỡ hoặc remove volume từ server</td>
</tr>
<tr>
<td>volume list</td>
<td>hiển thị danh sách các volume</td>
</tr>
<tr>
<td>volume show</td>
<td>Hiển thị thông tin chi tiết về volume</td>
</tr>
<tr>
<td>snapshot create</td>
<td>Tạo mới snapshot</td>
</tr>
<tr>
<td>snapshot delete</td>
<td>Xóa snapshot</td>
</tr>
<tr>
<td>snapshot list</td>
<td>Liệt kê danh sách snapshot</td>
</tr>
<tr>
<td>snapshot show</td>
<td>Hiển thị thông tin chi tiết về snapshot</td>
</tr>
<tr>
<td>volume type create</td>
<td>Tạo mới loại volume</td>
</tr>
<tr>
<td>volume type delete</td>
<td>Xóa flavor</td>
</tr>
<tr>
<td>volume type list</td>
<td>Hiển thi các loại volume đang hỗ trợ</td>
</tr></tbody></table>
