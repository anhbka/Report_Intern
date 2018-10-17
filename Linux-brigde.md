<h3>Linux-bridge</h3>
<h4>1. Khái niệm - ứng dụng </h4>
<p>Linux bridge là một phần mềm đươc tích hợp vào trong nhân Linux để giải quyết vấn đề ảo hóa phần network trong các máy vật lý. Về mặt logic Linux bridge sẽ tạo ra một switch ảo để cho các VM kết nối được vào và có thể nói chuyện được với nhau cũng như sử dụng để ra mạng ngoài</p>
<h4>2.Các thành phần <h4>
<img src="https://github.com/anhict/images/blob/master/lnb.png">
<h4>  Kiến trúc linux bridge minh họa như hình vẽ trên. Một số khái niệm liên quan tới linux bridge:</h4>
<p>-Port: tương đương với port của switch thật</p>
<p>-Bridge: tương đương với switch layer 2 </p>
<p>-Tap: hay tap interface có thể hiểu là giao diện mạng để các VM kết nối với bridge cho linux bridge tạo ra </p>
<p>-fd: forward data - chuyển tiếp dữ liệu từ máy ảo tới bridge </p>
<h4>  Các tính năng </h4>
<ul>
<li>STP: Spanning Tree Protocol - giao thức chống loop gói tin trong mạng</li>
<li>VLAN: chia switch (do linux bridge tạo ra) thành các mạng LAN ảo, cô lập traffic giữa các VM trên các VLAN khác nhau của cùng một switch.</li>
<li>FDB: chuyển tiếp các gói tin theo database để nâng cao hiệu năng switch</li>
</ul>
<h4>3. Lab </h4>  
<p>Một máy Ubuntu có cài KVM  card mạng là eth0 và eth1. Việc cài đặt KVM có thể làm theo hướng dẫn:</p>
<pre>https://www.server-world.info/en/note?os=Ubuntu_14.04&p=kvm&f=1</pre>
<h4> Mô hình lab </h4>
<img src="https://github.com/anhict/images/blob/master/Screenshot_4.png">
<p>Trường hợp 1: Tạo ra một bridge và gán port eth1 cho bridge đó ( sử dụng câu lệnh brctl ) </p>
<p> Bước 1: Tạo một bridge có tên là <code>linux</code></p>
<p>Bước 2: Gán port eth1 cho bridge đó</p>
<pre># brctl addif <tên bridge> <tên port gán>
VD: # brctl addif linux eth1</pre>
<p>Bước 3: Sửa lại file cấu hình /etc/network/interface </p>
<pre>auto linux
iface linux inet dhcp
bridge_ports eth1
bridge_stp off # kich hoat che do STP trong bridge
bridge_fd 0 
bridge_maxwait 0</pre>
<p>Bước 4: Khởi động lại internet</p>
<pre>ifdown -a && ifup -a</pre>
<p>Bước 5: Kiếm tra lại hoạt động của bridge</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_6.png">
<p>Cài đăt máy ảo chỉnh phần mạng chọn bridge linux</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_5.png">
<img src="https://github.com/anhict/images/blob/master/Screenshot_3.png">
  
  
