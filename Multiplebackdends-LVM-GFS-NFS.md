<h2>Cinder multiple backends (LVM, NFS, GlusterFS)</h2>
<h3>1. Mô hình </h3>
<img src="https://github.com/anhict/images/blob/master/Screenshot_53.png">
<p>Chú ý: Ở đây mình chỉ dùng 1 dải IP các bạn có thể tham khảo dùng nhiều dải IP theo mô hình https://github.com/anhict/images/blob/master/multiple_backends-mohinh.png</p>
<h3>2.Cài đặt </h3>
<ul>
<li>
<p>Lưu ý : Trên mô hình phải được cài đặt sẵn OpenStack và Cinder, nếu chưa có tham khảo tại <a href="https://github.com/congto/OpenStack-Mitaka-Scripts/tree/master/OPS-Mitaka-LB-Ubuntu">đây</a></p>
</li>
<li>
<p>Với mô hình trên LVM đã được cài đặt và thiết lập trước đó, ở bài này chỉ giới thiệu về glusterfs và nfs.</p>
</li>
<li>
<p>Xem chi tiết thiết lập Cinder và LVM tại <a href="https://github.com/hocchudong/ghichep-OpenStack/blob/master/05-Cinder/docs/cinder-install.md">đây</a></p>
</li>
</ul>
<h4>2.1 Trên node Cinder </h4>
<li>Cài đặt các gói <code>nfs-common</code> và <code>glusterfs-client</code> :</li>
<pre>apt-get -y install nfs-common glusterfs-client </pre>
<p>Chỉnh sửa file vi /etc/cinder/cinder.conf </p>
<p>Tại section [default] tìm và sửa dòng <code>enabled_backends</code> như sau :</p>
<pre>enabled_backends = lvm,nfs,glusterfs</pre>
<p>Thêm section nfs với nội dung như sau :</p>
<pre>[nfs]
volume_driver = cinder.volume.drivers.nfs.NfsDriver
volume_backend_name = NFS
nfs_shares_config = /etc/cinder/nfs_shares
nfs_mount_point_base = <span class="pl-smi">$state_path</span>/mnt_nfs </pre>
<p>Thêm section [glusterfs] với nội dung như sau :</p>
<pre>[glusterfs]
volume_driver = cinder.volume.drivers.glusterfs.GlusterfsDriver
volume_backend_name = GlusterFS
glusterfs_shares_config = /etc/cinder/glusterfs_shares
glusterfs_mount_point_base = <span class="pl-smi">$state_path</span>/mnt_glusterfs</pre>
<ul>
<li>Dùng trình soạn thảo <code>vi</code> mở file <code>/etc/cinder/glusterfs_shares</code> :</li>
</ul>
<p>Thêm vào nội dung :</p>
<div class="highlight highlight-source-shell"><pre><span class="pl-c"><span class="pl-c">#</span> create new : specify GlusterFS volumes</span>

192.168.239.196:/testvol2

<span class="pl-c"><span class="pl-c">#</span> testvol2 là trên của volume mà chúng ta sẽ tạo tạo GFS server.</span></pre></div>

<ul>
<li>Dùng trình soạn thảo <code>vi</code> mở file <code>/etc/cinder/nfs_shares</code> :</li>
</ul>

<p>Thêm vào nội dung :</p>
<pre>192.168.239.198:/nfsshare
<span class="pl-c"><span class="pl-c">#</span> /nfsshare là thư mục mà chúng ta sẽ share khi cấu hình NFS.</span></pre>
<ul>
<li>Thực hiện phân quyền và restart lại dịch vụ :</li>
</ul>
<pre>chmod 640 /etc/cinder/nfs_shares /etc/cinder/glusterfs_shares 
chgrp cinder /etc/cinder/nfs_shares /etc/cinder/glusterfs_shares 
service cinder-volume restart</pre>
<h4>2.2 Trên node compute.</h4>
<ul>
<li>Cài đặt các gói <code>nfs-common</code> và <code>glusterfs-client</code> :</li>
</ul>
<pre>apt-get -y install nfs-common glusterfs-client </pre>
<ul>
<li>Dùng trình soạn thảo <code>vi</code> chỉnh sửa file cấu hình nova :<pre>vi /etc/nova/nova.conf </pre></li>
</ul>
<ul>
<li>Tại section default thêm 2 dòng sau :</li>
</ul>
<pre>osapi_volume_listen = 0.0.0.0
volume_api_class = nova.volume.cinder.API</pre>

<ul>
<li>Restart lại dịch vụ :<code>service nova-compute restart</code></li>
</ul>
<h4>2.3 Trên Node gfs và gfs1 (thực hiện tương tự) :</h4>
<ul>
<li>Dùng trình soạn thảo <code>vi</code> để mở file hosts :</li>
</ul>
<pre>vi /etc/hosts</pre>
<ul>
<li>Sửa lại file host như sau :</li>
</ul>
<pre>192.168.239.190 controller
192.168.239.191 compute1
192.168.239.192 compute2
192.168.239.193 compute3
192.168.239.194 cinder
192.168.239.195	gfs-client
192.168.239.196	gfs
192.168.239.197	gfs1
192.168.239.198	nfs </pre>
<ul>
<li>Reboot lại máy chủ để lấy cấu hình mới nhất :<code>init 6</code></li>
</ul>

<ul>
<li>Thực hiện phân vùng ổ cứng :</li>
</ul>
<pre>fdisk /dev/sdb</pre>
<pre>root@lvm:~# fdisk /dev/sdb

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
Using default value 1
First sector (2048-62914559, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-62914559, default 62914559): +10G

Command (m for help): w
</pre>
<ul>
<li>Tải gói định dạng xfs :</li>
</ul>
<pre>apt-get install xfsprogs -y</pre>
<ul>
<li>Format phân vùng vừa tạo với định dạng xfs :</li>
</ul>
<pre>mkfs.xfs /dev/sdb1</pre>
<ul>
<li>Mount partition vào thư mục /mnt và tạo thư mục /mnt/brick1</li>
</ul>
<pre>mount /dev/sdb1 /mnt <span class="pl-k">&amp;&amp;</span> mkdir -p /mnt/brick1</pre>
<ul>
<li>Khai báo vào file cấu hình /etc/fstab để khi restart server, hệ thống sẽ tự động mount vào thư mục:</li>
</ul>
<ul>
<li>Cài đặt gói <code>glusterfs-server</code></li>
</ul>
<pre>apt-get install glusterfs-server</pre>

<h4>Tiếp tục thực hiện trên gfs1</h4>

<ul>
<li>Tạo 1 pool storage với Server gfs:</li>
</ul>
<pre>gluster peer probe gfs</pre>
<ul>
<li>Kiểm tra trạng thái của gluster pool</li>
</ul>
<pre>gluster peer status</pre>
<pre>root@gfs1:~# gluster peer status
Number of Peers: 1

Hostname: gfs
Uuid: 88c6034a-9135-45ac-85b1-7793d06ecced
State: Peer in Cluster (Connected)
root@gfs1:~#</pre>

<ul>
<li>Khởi tạo volume :</li>
</ul>
<pre>gluster  volume create testvol2 replica 2 transport tcp  gfs:/mnt/brick1  gfs1:/mnt/brick1</pre>
<ul>
<li>Start volume :</li>
</ul>
<pre>gluster volume start testvol2</pre>
<h4>2.4 Trên node gfs-client.</h4>
<ul>
<li>Dùng trình soạn thảo <code>vi</code> để mở file hosts :</li>
</ul>
<ul>
<li>Sửa lại file host như sau :</li>
</ul>
<pre>192.168.239.190 controller
192.168.239.191 compute1
192.168.239.192 compute2
192.168.239.193 compute3
192.168.239.194 cinder
192.168.239.195	gfs-client
192.168.239.196	gfs
192.168.239.197	gfs1
192.168.239.198	nfs </pre>
<ul>
<li>Cài đặt gói <code>glusterfs-client</code> :</li>
</ul>
<pre>glusterfs-client </pre>
<ul>
<li>Thực hiện mount từ thư mục tạo trên gfs server :</li>
</ul>
<pre>mount -t glusterfs 192.168.239.196:/testvol2 /mnt </pre>
<pre>
root@gfs-client:~# df -h
Filesystem                        Size  Used Avail Use% Mounted on
udev                              468M     0  468M   0% /dev
tmpfs                              98M  4.6M   93M   5% /run
/dev/mapper/gfs--client--vg-root   38G  1.6G   35G   5% /
tmpfs                             488M     0  488M   0% /dev/shm
tmpfs                             5.0M     0  5.0M   0% /run/lock
tmpfs                             488M     0  488M   0% /sys/fs/cgroup
/dev/sda1                         472M   58M  390M  13% /boot
tmpfs                              98M     0   98M   0% /run/user/0
192.168.239.196:/testvol2          20G   33M   20G   1% /mnt
root@gfs-client:~#</pre>

<h3>2.5 Cài đặt NFS </h3>
<ul>
<li>Chỉnh sửa file hosts:</li>
</ul>
<pre>192.168.239.190 controller
192.168.239.191 compute1
192.168.239.192 compute2
192.168.239.193 compute3
192.168.239.194 cinder
192.168.239.195	gfs-client
192.168.239.196	gfs
192.168.239.197	gfs1
192.168.239.198	nfs </pre>
<ul>
<li>Thực hiện phân vùng ổ cứng :</li>
</ul>
<pre>fdisk /dev/sdb</pre>
<pre>root@lvm:~# fdisk /dev/sdb

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
Using default value 1
First sector (2048-62914559, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-62914559, default 62914559): +10G

Command (m for help): w
</pre>
<li>Tải gói định dạng xfs :</li>
<pre>apt-get install xfsprogs -y</pre>
<ul>
<li>Format phân vùng vừa tạo với định dạng xfs :</li>
</ul>
<pre>mkfs.xfs /dev/sdb1</pre>
<ul>
<li>Mount partition vào thư mục /mnt :</li>
</ul>
<pre>mount /dev/sdb1 /mnt</pre>

<ul>
<li>Tạo thư mục /mnt/nfs :</li>
</ul>
<pre>mkdir -p /mnt/nfs</pre>

<ul>
<li>Cài đặt gói <code>nfs-kernel-server</code> :</li>
</ul>
<pre>apt-get -y install nfs-kernel-server</pre>

<ul>
<li>Thực hiện lệnh</li>
</ul>
<div class="highlight highlight-source-shell"><pre><span class="pl-c1">echo</span> <span class="pl-s"><span class="pl-pds">"</span>/mnt/nfs 192.168.239.196/24(rw,no_root_squash)<span class="pl-pds">"</span></span> <span class="pl-k">&gt;&gt;</span> /ect/exports</pre></div>
<p>Cho phép mount thư mục /mnt/nfs đến dải 192.168.239.0/24</p>
<ul>
<li>Thực hiện lệnh :</li>
</ul>
<div class="highlight highlight-source-shell"><pre><span class="pl-c1">echo</span> <span class="pl-s"><span class="pl-pds">"</span>/dev/sdb1 /mnt xfs defaults 0 0<span class="pl-pds">"</span></span> <span class="pl-k">&gt;&gt;</span> /etc/fstab</pre></div>
<p>Để hệ thống tự động mount vào thư mục khi restart hệ thống.</p>
<pre>service nfs-kernel-server restart</pre>
<h3>2.6 Cài đặt trên node nfs-client (Chỉ mang mục đích thử nghiệm quá trình mount đã thành công hay chưa).</h3>
<p>Chỉnh sửa file hosts:</p>
<pre>192.168.239.190 controller
192.168.239.191 compute1
192.168.239.192 compute2
192.168.239.193 compute3
192.168.239.194 cinder
192.168.239.195	gfs-client
192.168.239.196	gfs
192.168.239.197	gfs1
192.168.239.198	nfs </pre>
<p>Cài đặt gói nfs-common :</p>
<pre>apt-get -y install nfs-common</pre>
<p>Tạo thư mục /mnt/a để mount thư mục từ nfs server về :</p>
<pre>mkdir -p /mnt/a</pre>
<p>Thực hiện mount thư mục mnt/nfs về thư mục /mnt/a :<p>
<pre>mount 192.168.239.198:/mnt/nfs /mnt/a</pre>
<h4>2.7  Cài đặt trên node Cinder.</h4>
<p>Thực hiện lệnh :</p>
<pre>echo "/dev/sdb1 /mnt xfs defaults 0 0" >> /etc/fstab</pre>
<p>Để hệ thống tự động mount vào thư mục khi restart hệ thống.</p>
<h3>3. Kiểm tra </h3>
<p>Trên node controller , chúng ta thực hiện cấu hình backends :<p>
<pre>root@controller:~# . admin-openrc
root@controller:~# cinder service-list
+------------------+-------------+------+---------+-------+----------------------------+-----------------+
| Binary           | Host        | Zone | Status  | State | Updated_at                 | Disabled Reason |
+------------------+-------------+------+---------+-------+----------------------------+-----------------+
| cinder-scheduler | controller  | nova | enabled | up    | 2018-07-15T13:46:03.000000 | -               |
| cinder-volume    | cinder@gfs  | nova | enabled | up    | 2018-07-16T04:09:04.000000 | -               |
| cinder-volume    | cinder1@lvm | nova | enabled | up    | 2018-07-15T13:44:23.000000 | -               |
| cinder-volume    | cinder@nfs  | nova | enabled | up    | 2018-07-15T10:01:18.000000 | -               |
+------------------+-------------+------+---------+-------+----------------------------+-----------------+
</pre>

<p>Tạo volume-type cho glusterfs :</p>

<pre>cinder type-create glusterfs
cinder type-create nfs</pre>
<pre>root@controller:~# cinder type-create glusterfs
+--------------------------------------+-----------+-------------+-----------+
| ID                                   | Name      | Description | Is_Public |
+--------------------------------------+-----------+-------------+-----------+
| 94fbd50d-f1f7-47cf-a6e8-dd1b6c4f4912 | glusterfs | -           | True      |
+--------------------------------------+-----------+-------------+-----------+
root@controller:~# cinder type-create nfs
+--------------------------------------+------+-------------+-----------+
| ID                                   | Name | Description | Is_Public |
+--------------------------------------+------+-------------+-----------+
| 2f697eb1-1df9-46ee-97c9-509921483750 | nfs  | -           | True      |
+--------------------------------------+------+-------------+-----------+</pre>
<p>Tạo một volume có backends gfs để thử nghiệm :</p>
<pre>root@controller:~# cinder create --display_name disk01 10 --volume-type glusterfs
+--------------------------------+--------------------------------------+
| Property                       | Value                                |
+--------------------------------+--------------------------------------+
| attachments                    | []                                   |
| availability_zone              | nova                                 |
| bootable                       | false                                |
| consistencygroup_id            | None                                 |
| created_at                     | 2018-07-14T14:16:25.000000           |
| description                    | None                                 |
| encrypted                      | False                                |
| id                             | 47098204-aaa0-4ef3-9ae4-23669b00292c |
| metadata                       | {}                                   |
| migration_status               | None                                 |
| multiattach                    | False                                |
| name                           | disk01                               |
| os-vol-host-attr:host          | cinder@lvm#LVM                       |
| os-vol-mig-status-attr:migstat | None                                 |
| os-vol-mig-status-attr:name_id | None                                 |
| os-vol-tenant-attr:tenant_id   | 56b787b228b64536adacbff5e23fd321     |
| replication_status             | None                                 |
| size                           | 10                                   |
| snapshot_id                    | None                                 |
| source_volid                   | None                                 |
| status                         | available                            |
| updated_at                     | 2018-07-14T14:16:11.000000           |
| user_id                        | 9e724120e03d43f795109d47fabd5ff5     |
| volume_type                    | glusterfs                            |
+--------------------------------+--------------------------------------+
root@controller:~#</pre>
<p>Kiểm tra tình trạng của volume : <code>cinder list</code></p>
<pre>root@controller:~# cinder list
+--------------------------------------+-----------+--------+------+-------------+----------+-------------+
| ID                                   | Status    | Name   | Size | Volume Type | Bootable | Attached to |
+--------------------------------------+-----------+--------+------+-------------+----------+-------------+
| 47098204-aaa0-4ef3-9ae4-23669b00292c | available | disk01 | 10   | glusterfs   | false    |             |
+--------------------------------------+-----------+--------+------+-------------+----------+-------------+</pre>

<p>Tạo một volume có backends nfs để thử nghiệm :</p>
<pre>root@controller:~# cinder create --display_name disk_nfs-2 --volume-type nfs 10
+--------------------------------+--------------------------------------+
| Property                       | Value                                |
+--------------------------------+--------------------------------------+
| attachments                    | []                                   |
| availability_zone              | nova                                 |
| bootable                       | false                                |
| consistencygroup_id            | None                                 |
| created_at                     | 2018-07-14T14:38:01.000000           |
| description                    | None                                 |
| encrypted                      | False                                |
| id                             | 670f0c57-2502-4aa5-b444-f19f94a86693 |
| metadata                       | {}                                   |
| migration_status               | None                                 |
| multiattach                    | False                                |
| name                           | disk_nfs-2                           |
| os-vol-host-attr:host          | cinder@lvm#LVM                       |
| os-vol-mig-status-attr:migstat | None                                 |
| os-vol-mig-status-attr:name_id | None                                 |
| os-vol-tenant-attr:tenant_id   | 56b787b228b64536adacbff5e23fd321     |
| replication_status             | None                                 |
| size                           | 10                                   |
| snapshot_id                    | None                                 |
| source_volid                   | None                                 |
| status                         | creating                             |
| updated_at                     | 2018-07-14T14:38:02.000000           |
| user_id                        | 9e724120e03d43f795109d47fabd5ff5     |
| volume_type                    | nfs                                  |
+--------------------------------+--------------------------------------+</pre>
<p>Kiểm tra:</p>
<pre>root@controller:~# cinder list
+--------------------------------------+-----------+------------+------+-------------+----------+-------------+
| ID                                   | Status    | Name       | Size | Volume Type | Bootable | Attached to |
+--------------------------------------+-----------+------------+------+-------------+----------+-------------+
| 47098204-aaa0-4ef3-9ae4-23669b00292c | available | disk01     | 10   | glusterfs   | false    |             |
| 670f0c57-2502-4aa5-b444-f19f94a86693 | available | disk_nfs-2 | 10   | nfs         | false    |             |
+--------------------------------------+-----------+------------+------+-------------+----------+-------------+
root@controller:~#</pre>











