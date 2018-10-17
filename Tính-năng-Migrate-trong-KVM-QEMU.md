<h2>Tài liệu lab Live Migrate trên KVM</h2>
<h3>1. Khái niệm KVM-QEMU </h3>
<p>KVM (Kernel-base virtual machine): là một mudule nằm trong nhân Linux để có thể tạo ra không gian cho các ứng dụng để các ứng dụng đó có thể chạy các tính với quyền lớn nhất. Qemu: là một hypervisor dạng Paravirtualization nó tương tự như VMwate workstation</p>
<p>Vậy KVM-QEMU là ảo hóa kết hợp QEMU với KVM theo kiểu QEMU sẽ móc nối với mudule KVM trở thành dạng ảo hóa Full virtualization</p>
<p>Kiến trúc của KVM-QEMU</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f4c44554a534e5a2e706e67.png">
<h3>2. Các tool để điều khiển KVM-Qemu</h3>
<ul>
<li>Bao gồm:</li>

<li>virsh</li>
<li>virt-manager</li>
<li>Openstack</li>
<li>ovirt</li>
...</ul>
<p>Hình trên ta có thể hiểu: Đối với từng dạng ảo hóa như Kvm, Xen, .. sẽ có một tiến trình Libvirt chạy để điều khiển các dang ảo hóa và cung cấp những API để các tool như virsh, virt-manager, Openstack, ovirt có thể giao tiếp với KVM-Qemu thông qua livbirt</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f6332516e3456382e706e67.png">
<h3>3. Tính năng Migrate trong KVM-QEMU</h3>
<h4>Khái niệm</h4>
<p>Migrate là chức năng được KVM-QEMU hỗ trợ, nó cho phép di chuyển các guest từ một host vật lý này sang host vật lý khác và không ảnh hướng để guest đang chạy cũng như dữ liệu bên trong nó.</p>
<h4>Vai trò</h4>
<p>Migrate giúp cho nhà quản trị có thể di chuyển các guest trên host đi để phục vụ cho việc bảo trì và nâng cấp hệ thống, nó cũng giúp nhà quản trị nâng cao tính dự phòng, và cũng có thể làm nhiệm vụ load balancing cho hệ thống khi một máy host quá tải.</p>
<h4>Cơ chế:</h4>
<p>Migrate có 2 cơ chế:</p>
<li>Cơ chế Offline Migrate: là cơ chế cần phải tắt guest đi thực hiện việc di chuyển image và file xml của guest sang một host khác Mô hình thuần túy của cơ chế Offline Migrate.</li>
<li>Cơ chế Live Migrate: đây là cơ chế di chuyển guest khi guest vẫn đang hoạt động, quá trình trao đổi diễn ra rất nhanh các phiên làm việc kết nối hầu như không cảm nhận được sự gián đoạn nào. Quá trình Live Migrate được diễn ra như sau: Bước đầu tiên của quá trình Live Migrate 1 ảnh chụp ban đầu của guest trên host1 được chuyển sang host2. Trong trường hợp người dùng đang truy cập tại host1 thì những sự thay đổi và hoạt động trên host1 vẫn diễn ra bình thường, tuy nhiên những thay đổi này sẽ được ghi nhận. Những thay đổi trên host1 được đồng bộ liên tục đến host2 Khi đã đồng bộ xong thì guest trên host1 sẽ offline và các phiên truy cập trên host1 được chuyển sang host2.</li>
<h3>LAB</h3>
<p>Mô hình </p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_18.png">
<p>Thực hiện tính năng Migrate đối với cơ chế Live Migrate kết hợp với hệ thống chia sẻ file NFS</p>
<p>Ý tưởng của cơ chế này: Cần một Server Storage chia sẻ một thư mục để 2 host có thể móc mount vào thư mục đó</p>
<h4>Cài đặt</h4>
<p>1: Xây dựng mô hình như trong hình vẽ</p>
<p>2: Cài đặt 2 host chạy KVM-QEMU và NFS</p>
<p>Cấu hình cho phép giao thức TCP trong dịch vụ libvirt</p>
<p>Sửa file /etc/libvirt/libvirtd.conf</p>
<pre>listen_tls = 0  
listen_tcp = 1
tcp_port = "16509"
listen_addr = "0.0.0.0"
auth_tcp = "none"
mdns_adv = 0</pre>
<p>Giải thích</p>
<pre>- Listen_tls : tắt tls, mặc định nó được mở.
- listen_tcp: bật chức năng kiểm duyệt tcp
- tcp_port: cấu hình cổng tcp, mặc định là 16509
- auth_tcp: bật hoặc tắt việc kiểm duyệt bằng mật khẩu
- mdns_adv: bật/tắt tính năng mdns multicast, mặc định là tắt.</pre>
<p>Cấu hình iptables cho phép giao thông đi qua các cổng 16509 và 49152:</p>
<pre>iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 5900:5909 -j ACCEPT
iptables -A INPUT -p tcp --dport 16509 -j ACCEPT
iptables -A INPUT -p tcp --dport 49152 -j ACCEPT</pre>
<p>Nếu bạn sử dụng quyền root để migrate thì sửa file sau để nói cho hệ thống biết:</p>
<p>vi /etc/libvirt/qemu.conf bỏ comment #user = "root" và #group= "root"</p>
<p>Cài đặt KVM-QEMU:</p>
<p>aptitude -y install qemu-kvm libvirt-bin virtinst bridge-utils</p>
<p>Kích hoạt chế độ tạo vhost-net</p>
<pre>modprobe vhost_net 
 lsmod | grep vhost
echo vhost_net >> /etc/modules</pre>
Chỉnh sửa lại file interface để cấu hình Brigde network như sau
<pre>vi /etc/network/interface
The loopback network interface
auto lo
iface lo inet loopback
auto br0
iface br0 inet dhcp
bridge_ports eth0
bridge_fd 9
bridge_hello 2
bridge_maxage 12
bridge_stp off
auto eth0
iface eth0 inet manual
up ip link set dev $IFACE up</pre>
<p>Cài đặt virt-manager với mục đích quan sát</p>
<pre>aptitude -y install virt-manager qemu-system hal</pre>
<p>Cài đặt NFS client theo link: http://www.server-world.info/en/note?os=Ubuntu_14.04&p=nfs&f=2 </p>
<p>Cài đặt NFS server theo link: http://www.server-world.info/en/note?os=Ubuntu_14.04&p=nfs&f=1 </p>
<p>Đầu tiên tạo 1 máy ảo trên KVM có tên VM01 sử dụng image linux-microcore-3.8.2.img sau đó tiến hành migrate máy ảo</p>
<p>Đầu tiên, muốn migrate máy ảo sang kvm11 cần tạo một file y hệt với file cấu hình của máy ảo trên kvm22</p>
<code>qemu-img create -f qcow2 -o preallocation=metadata linux-microcore-3.8.2.img.qcow2 24444928</code>
<h4>Migrate dùng câu lệnh virsh</h4>
<p>Thực hiện câu lênh trên host1 ( chứa VM01 đang chạy )</p>
<code>virsh migrate --live <tên guest muốn migrate> qemu+ssh://<hostname của đích chuyển đến>/system</code>
<code>Ví dụ: virsh migrate --live VM01 qemu+ssh://192.168.239.209/system</code>
<p>Kiểm tra lại: </p>
<pre>root@kvm11:~# virsh migrate --live VM01 qemu+ssh://192.168.239.209/system
root@192.168.239.209's password:
root@kvm22:/var/lib/libvirt/images# virsh list
 Id    Name                           State
----------------------------------------------------
 9     VM01                           running</pre>
 <p>Khi đó quan sát trên virt-manager sẽ thấy VM01 trên KVM11 sẽ di chuyển sang KVM22 mà vẫn đang ở trạng thái hoạt động thời gian downtime của VM01 là khá nhỏ </p>

 <p>Ngoài tùy chọn --live. Còn thêm một số các tùy chọn khác như: </p>

 <p>--live : tùy chọn chuyển trực tiếp guest khi đang chạy mà không làm tắt guest ( nếu không có tùy chọn này thì guest sẽ khởi động lại ) </p>

 <p>--persistent : tùy chọn chuyển máy ảo sang host mới mà khi tắt guest đó, guest sẽ không bị mất </p>

 <p>--undefinesource : tùy chọn này sẽ xóa guest ở nguồn đi </p>

 <p>--suspend : tùy chọn sẽ tạm dừng máy ảo khi chuyển sang máy host mới </p>

 <p>--unsafe : chuyển máy ngay cả trong chế độ không an toàn </p>

 <p>Ngoài những tùy chọn trên có 3 tùy chọn rất đặc biệt, cũng có thể nói là tính năng nâng cao hơn đối với việc Migrate </p>

 <p>--direct : Sử dụng migrate trực tiếp host mà không cần 2 host đó phải sử dụng chung thư mục </p>

 <p>--p2p : Sử dụng cho việc Migrate peer-to-peer </p>

--tunnelled : Sử dụng cơ chế tunnel để migrate các guest  </p>
 
 <img src="https://github.com/anhict/images/blob/master/Screenshot_19.png">
 <img src="https://github.com/anhict/images/blob/master/Screenshot_20.png">
 <img src="https://github.com/anhict/images/blob/master/Screenshot_21.png">
