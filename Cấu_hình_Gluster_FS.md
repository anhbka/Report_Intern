<h3>1. Giới thiệu Gluster FS</h3>
<h4>1.1 Giới thiệu Gluster FS</h4>
<ul>
<li>
<p>GlusterFS là một open source, là tập hợp file hệ thống có thể được nhân rộng tới vài peta-byte và có thể xử lý hàng ngàn Client.</p>
</li>
<li>
<p>GlusterFS có thể linh hoạt kết hợp với các thiết bị lưu trữ vật lý, ảo, và tài nguyên điện toán đám mây để cung cấp 1 hệ thống lưu trữ có tính sẵn sàng cao và khả năng performant cao .</p>
</li>
<li>
<p>Chương trình có thể lưu trữ dữ liệu trên các mô hình, thiết bị khác nhau, nó kết nối với tất cả các nút cài đặt GlusterFS qua giao thức TCP hoặc RDMA tạo ra một nguồn tài nguyên lưu trữ duy nhất kết hợp tất cả các không gian lưu trữ có sẵn thành một khối lượng lưu trữ duy nhất (distributed mode) hoặc sử dụng tối đa không gian ổ cứng có sẵn trên tất cả các ghi chú để nhân bản dữ liệu của bạn (replicated mode).</p>
</li>
</ul>
<h4>1.2 Thuật ngữ trong Gluster FS</h4>
<p>Để có thể hiểu rõ về GlusterFS và ứng dụng được sản phẩm này, trước hết ta cần phải biết rõ những khái niệm có trong GlusterFS. Sau đây là những khái niệm quan trọng khi sử dụng Glusterfs</p>
<ul>
<li><b>Trusted Storage Pool</b>: Trong một hệ thống GlusterFS, những server dùng để lưu trữ được gọi là những node, và những node này kết hợp lại với nhau thành một không gian lưu trữ lớn được gọi là Pool. Dưới đây là mô hình kết nối giữa 2 node thành một Trusted Storage Pool.</li>
</ul>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f57757842536e5a2e706e67.png">
<ul>
<li><b>Brick</b>:</li>
</ul>
<ul>
<li>Từ những phần vùng lưu trữ mới (những phân vùng chưa dùng đến) trên mỗi node, chúng ta có thể tạo ra những brick.</li>
<li>Brick được định nghĩa bởi 1 server (name or IP) và 1 đường dẫn. Vd: 10.10.10.20:/mnt/brick (đã mount 1 partition (/dev/sdb1) vào /mnt)</li>
<li>Mỗi brick có dung lượng bị giới hạn bởi filesystem....</li>
<li>Trong mô hình lý tưởng, mỗi brick thuộc cluster có dung lượng bằng nhau. Để có thể hiểu rõ hơn về Bricks, chúng ta có thể tham khảo hình dưới đây:</li>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f7645766d304a372e706e67.png"> 
</ul>
<ul>
<li><b>Volume</b>:</li>
</ul>
<ul>
<li>Từ những brick trên các node thuộc cùng một Pool, kết hợp những brick đó lại thành một không gian lưu trữ lớn và thống nhất để client có thể mount đến và sử dụng.</li>
<li>Một volume là tập hợp logic của các brick. Tên volume được chỉ định bởi administrator</li>
<li>Volume được mount bởi client: mount -t glusterfs server1:/ /my/mnt/point </li>
<li>Một volume có thể chứa các brick từ các node khác nhau. Sau đây là mô hình tập hợp những Brick thành Volume:</li>
</ul>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f53676f6c5654712e706e67.png">
<p>Tại hình trên, chúng ta có thể thấy mỗi Node1, Node2, Node3 đã tạo 2 brick là /export/brick1 và /export/brick2, và từ 3 brick /export/brick1trên 3 Node tập hợp lại tạo thành volume music. Tương tự 3 brick /export/brick2 trên 3 Node tập hợp lại tạo thành volume Videos.</p>
<h3>1.3 Một số loại volume cơ bản</h3>
<p>Khi sử dụng GlusterFS có thể tạo nhiều loại volume và mỗi loại có được những tính năng khác nhau. Dưới đây là 5 loại volume cơ bản</p>
<strong>Distributed volume:</strong>
<p>Distributed Volume có những đặc điểm cơ bản sau:</p>
<p>Dữ liệu được lưu trữ phân tán trên từng bricks, file1 nằm trong brick 1, file 2 nằm trong brick 2,...</p>
<p>Vì metadata được lưu trữ trực tiếp trên từng bricks nên không cần thiết phải có một metadata server ở bên ngoài, giúp cho các tổ chức tiết kiệm được tài nguyên.</p>
<p>Ưu điểm: mở rộng được dung lượng store ( dung lượng store bằng tổng dung lượng các brick)</p>
<p>Nhược điểm: nếu 1 trong các brick bị lỗi, dữ liệu trên brick đó sẽ mất</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f5a41366438664f2e706e67.png">
<strong>Replicated volume:</strong>
<p>Dữ liệu sẽ được nhân bản đến những brick còn lại, trên tất cả các node và đồng bộ tất cả các nhân bản mới cập nhật.</p>
<p>Đảm bảo tính nhất quán.</p>
<p>Không giới hạn số lượng replicas.</p>
<p>Ưu điểm: phù hợp với hệ thống yêu cầu tính sẵn sàng cao và dự phòng</p>
<p>Nhược điểm: tốn tài nguyên hệ thống</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f48396d73424e482e706e67.png">
<strong>Stripe volume:</strong>
<p>Dữ liệu chia thành những phần khác nhau và lưu trữ ở những brick khác nhau, ( 1 file được chia nhỏ ra trên các brick )</p>
<p>Ưu điểm : phù hợp với những môi trường yêu cầu hiệu năng, đặc biệt truy cập những file lớn.</p>
<p>Nhược điểm: 1 brick bị lỗi volume không thể hoạt động được.</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f6e50496e59656e2e706e67.png">
<strong>Distributed replicated:</strong>
<p>Kết hợp từ distributed và replicated</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f62454f746753372e706e67.png">
<p>Với mô hình trên, hệ thống sẽ yêu cầu cần tối thiểu 3 node, vừa có thể mở rộng được dung lượng lưu trữ, vừa tăng tính dự phòng cho hệ thống. Tuy nhiên, nếu đồng thời bị lỗi 2 node server1 và server2 hoặc 2 node server3 và server4 thì hệ thống sẽ không hoạt động được.</p>
<strong>Distributed stripe volume:</strong>
<p>Kết hợp từ Distributed và stripe. Do đó nó có hầu hết những thuộc tính hai loại trên và khi 1 node và 1 brick delete đồng nghĩa volume cũng không thể hoạt động được nữa.</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f765236463761322e706e67.png">
<strong>Replicated stripe volume</strong>
<p>Kết hợp từ replicated và stripe</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f6e52696a754a792e706e67.png">
<h3>2. Các bước chuẩn bị dựng lab GlusterFS</h3>
<p>Sau đây là các bước chuẩn bị để dựng bài lab GlusterFS: IP Planing và Topology lab</p>
<strong>2.1 IP Planing</strong>
<table>
<thead>
<tr>
<th>Host</th>
<th>OS</th>
<th>IP</th>
<th>Disk 1</th>
<th>Disk 2</th>
<th>Hostname</th>
</tr>
</thead>
<tbody>
<tr>
<td>Server 01</td>
<td>Ubuntu 14</td>
<td>192.168.239.197</td>
<td>sda 20G cài OS</td>
<td>sdb 10G trống</td>
<td>gluster01</td>
</tr>
<tr>
<td>Server 02</td>
<td>Ubuntu 14</td>
<td>192.168.239.198</td>
<td>sda 20G cài OS</td>
<td>sdb 10G trống</td>
<td>gluster02</td>
</tr>
<tr>
<td>Client</td>
<td>Ubuntu 14</td>
<td>192.168.239.199</td>
<td>sda 20G cài OS</td>
<td>Không có</td>
<td>Client</td>
</tr></tbody></table>
<img src="https://github.com/anhict/images/blob/master/Screenshot_42.png">
<h3>3. Dựng lab Gluster</h3>
<p>Thực hiện các bước sau trên Server01</p>
<p>Bước 1: Khai báo trong file /etc/hosts như sau</p>
<pre>127.0.0.1       localhost
127.0.1.1       gluster01
192.168.239.247  gluster01
192.168.239.246  Client
192.168.239.248     gluster02</pre>
<p>Bước khai báo trên giúp các node trong hệ thống có thể ping đến nhau thông qua hostname</p>
<ul>
<li>Bước 2: Phân vùng cho ổ cứng</li>
</ul>
<pre>fdisk /dev/sdb</pre>
<ul>
<li>Bước 3: Format phân vùng định dạng xfs. Trong 1 số trường hợp không có sẵn gói định dạng thì phải tải về</li>
</ul>
<pre>apt-get install xfsprogs -y
mkfs.xfs /dev/sdb1</pre>
<ul>
<li>Bước 4: Mount partition vào thư mục /mnt và tạo thư mục /mnt/brick1</li>
</ul>
<pre>mount /dev/sdb1 /mnt && mkdir -p /mnt/brick1</pre>
<ul>
<li>Bước 5: Khai báo vào file cấu hình /etc/fstab để khi restart server, hệ thống sẽ tự động mount vào thư mục.</li>
</ul>
<pre>echo "/dev/sdb1 /mnt xfs defaults 0 0" >> /etc/fstab</pre>
<ul>
<li>Bước 6: Tải gói gluster server</li>
</ul>
<pre>apt-get install glusterfs-server -y</pre>
<strong>Thực hiện các bước sau trên Gluster02</strong>
<p>Phân vùng cho ổ cứng</p>
<pre>fdisk /dev/sdb</pre>
<p>Bước 3: Format phân vùng định dạng xfs. Trong 1 số trường hợp không có sẵn gói định dạng thì phải tải về</p>

<pre>apt-get install xfsprogs -y
mkfs.xfs /dev/sdb1</pre>
<p>Bước 4: Mount partition vào thư mục /mnt và tạo thư mục /mnt/brick1</p>
<pre>mount /dev/sdb1 /mnt && mkdir -p /mnt/brick1</pre>
<p>Bước 5: Khai báo vào file cấu hình /etc/fstab để khi restart server, hệ thống sẽ tự động mount vào thư mục.</p>
<pre>echo "/dev/sdb1 /mnt xfs defaults 0 0" >> /etc/fstab</pre>
<p>Bước 6: Tải gói gluster server</p>
<pre>apt-get install glusterfs-server -y</pre>
<p>Bước 7: Tạo 1 pool storage với Server gluster01</p>
<code>gluster peer probe 192.168.239.247</code>
<p>Bước 8: Kiểm tra trạng thái của gluster pool</p>
<pre>root@Gluster01:~# gluster peer status
Number of Peers: 1

Hostname: 192.168.239.248
Port: 24007
Uuid: cb5b087d-9df1-40f4-9913-a19eed0fe78e
State: Peer in Cluster (Connected)</pre>
<p>Như vậy tôi đã tạo 1 pool với 2 brick từ 2 node storage</p>
<ul>
<li>Bước 9: Sau đây tôi sẽ tạo 1 Volume Replicated</li>
</ul>
<pre>gluster volume create testvol2 rep 2 transport tcp 192.168.239.247:/mnt/brick1 192.168.239.248:/mnt/brick1
root@Gluster01:~# gluster volume create testvol2 rep 2 transport tcp 192.168.239.247:/mnt/brick1 192.168.239.248:/mnt/brick1
volume create: testvol2: success: please start the volume to access data
</pre>

<strong>Giải thích cú pháp lệnh:</strong>
<table>
<thead>
<tr>
<th>Cú pháp</th>
<th>Ý nghĩa</th>
</tr>
</thead>
<tbody>
<tr>
<td>gluster volume create</td>
<td>Câu lệnh để tạo volume</td>
</tr>
<tr>
<td>testvol2</td>
<td>Tên volume</td>
</tr>
<tr>
<td>rep</td>
<td>Loại volume là replicated</td>
</tr>
<tr>
<td>2</td>
<td>Số brick</td>
</tr>
<tr>
<td>transport tcp</td>
<td>Giao thức để liên lạc là tcp</td>
</tr>
<tr>
<td>192.168.239.197:/mnt/brick1</td>
<td>Đường dẫn của brick</td>
</tr>
<tr>
<td>192.168.239.198:/mnt/brick1</td>
<td>Đường dẫn của brick</td>
</tr></tbody></table>

<ul>
<li>Bước 10: Sau khi tạo volume, tôi sẽ khởi động volume đó lên. Bước này có thể thực hiện trên cả 2 Server gluster</li>
</ul>
<h3>3.2 Các bước cấu hình trên client</h3>
<p>Bước 1: Tải gói gluster client</p>
<pre>apt-get install glusterfs-client -y</pre>
<p>Bước 2: Mount volume về sử dụng</p>
<pre>mount -t glusterfs 192.168.239.197:/testvol2 /mnt
root@Client:~# mount -t glusterfs 192.168.239.247:/testvol2 /mnt
root@Client:~# df -hT
Filesystem                Type            Size  Used Avail Use% Mounted on
udev                      devtmpfs        981M  4.0K  981M   1% /dev
tmpfs                     tmpfs           199M  956K  198M   1% /run
/dev/dm-0                 ext4             18G  1.3G   16G   8% /
none                      tmpfs           4.0K     0  4.0K   0% /sys/fs/cgroup
none                      tmpfs           5.0M     0  5.0M   0% /run/lock
none                      tmpfs           992M     0  992M   0% /run/shm
none                      tmpfs           100M     0  100M   0% /run/user
/dev/sda1                 ext2            236M   40M  184M  18% /boot
192.168.239.247:/testvol2 fuse.glusterfs   10G   33M   10G   1% /mnt
root@Client:~#
</pre>
<p>Bước 3: Kiểm tra</p>
<pre>root@Client:~# df -hT
Filesystem                Type            Size  Used Avail Use% Mounted on
udev                      devtmpfs        981M  4.0K  981M   1% /dev
tmpfs                     tmpfs           199M  956K  198M   1% /run
/dev/dm-0                 ext4             18G  1.3G   16G   8% /
none                      tmpfs           4.0K     0  4.0K   0% /sys/fs/cgroup
none                      tmpfs           5.0M     0  5.0M   0% /run/lock
none                      tmpfs           992M     0  992M   0% /run/shm
none                      tmpfs           100M     0  100M   0% /run/user
/dev/sda1                 ext2            236M   40M  184M  18% /boot
192.168.239.247:/testvol2 fuse.glusterfs   10G   33M   10G   1% /mnt
root@Client:~#</pre>
<p>Như vậy là tôi đã đứng trên client và mount về được volume đã tạo trên gluster server</p>
<p>Sau đây tôi sẽ test thử tính năng của gluster</p>
<h3>3.3 Kiểm tra khả năng hoạt động của Gluster FS</h3>
<p>Để kiểm tra khả năng hoạt động của gluster tôi sẽ copy 1 file iso vào thư mục /mnt trên client tôi đã mount volume</p>
<pre>
root@Client:/mnt# ls
cirros-0.4.0-x86_64-disk.img</pre>
<p>Do ở đây tôi tạo replicated volume nên theo đặc tính của của loại volume này tôi sẽ nhìn thấy được dữ liệu cả ở trên 2 server gluster</p>
<p>Bước 2: Kiểm tra trên server gluster01</p>
<pre>root@Gluster01:/mnt/brick1# ls
cirros-0.4.0-x86_64-disk.img
root@Gluster01:/mnt/brick1#</pre>
<p>Bước 3: Kiểm tra trên server gluster02</p>
<pre>root@Gluster02:/mnt/brick1# ls
cirros-0.4.0-x86_64-disk.img
root@Gluster02:/mnt/brick1#</pre>
<pre>root@Gluster01:/mnt/brick1# gluster volume info

Volume Name: testvol2
Type: Replicate
Volume ID: 838c81b1-0bdf-4737-a524-28dc9de4bf1e
Status: Started
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: 192.168.239.247:/mnt/brick1
Brick2: 192.168.239.248:/mnt/brick1
root@Gluster01:/mnt/brick1#</pre>

<p>Như vậy là tôi đã dựng xong bài lab với GlusterFS sử dụng Replicated Volume.</p>











          










          

