<p>Tương tự như VMWare, các máy ảo của KVM cũng có 3 tùy chọn card mạng đó là NAT, Public Bridge và Private Bridge.</p>
<h3>1. NAT </h3>
<p>- Đây là cấu hình card mạng mặc định của KVM. Cơ chế NAT sẽ cấp cho mỗi VM một địa chỉ IP theo dải mặc định, nó sẽ hoạt động giống với một chiếc Router, chuyển tiếp các gói tin giữa những lớp mạng khác nhau trên một mạng lớn. Về mặt logic, ta có thể hiểu nó là 1 bridge riêng biệt và nó sẽ giao tiếp với bridge mà card mạng thật kết nối để các máy ảo có thể kết nối ra bên ngoài mạng internet. Chúng ta có thể xem dải mạng mặc định mà cơ chế NAT sẽ cấp cho các máy ảo ở trong file:</p>
<pre>/var/lib/libvirt/network/default.xml</pre>
<img src="https://github.com/anhict/images/blob/master/Screenshot_1.png">
<p>- Các card mạng của máy ảo sẽ được gắn vào 1 bridge mặc định (vibr0), bridge này đã có gateway mặc định, các gói tin của máy ảo sẽ đi qua bridge này trước khi được chuyển tới bridge có kết nối từ card mạng thật để ra ngoài internet.</p>
<p>- KVM sẽ cấp DHCP cho các máy dùng chế độ NAT theo dải mặc định, người dùng có thể xem tại file /var/lib/libvirt/network/default.xml </p>
<h3>2. Public Bridge</h3>
<p>- Chế độ này sẽ cho phép các máy ảo có cùng dải mạng vật lí với card mạng thật. Để có thể làm được điều này, bạn cần thiết lập 1 bridge và cho phép nó kết nối với cổng vật lí của thiết bị thật (eth0).</p>
<p>- Các bước để cấu hình public bridge:</p>
<ul>
<li>Sử dụng câu lệnh <code>brctl addbr Ten_bridge</code> để tạo một bridge</li>
<li>Gán card mạng thật cho bridge đó bằng câu lệnh <code>brctl addif Ten_bridge Ten_card_mang</code> .</li>
<li>Cấu hình cho card mạng trong file: <code>/etc/network/interfaces</code> . Tại đây, hãy comment các cấu hình của card mạng vật lý và cấu hình lại cho bridge vừa tạo</li>
<li>Khởi động lại dịch vụ mạng</li>
</ul>
<p>- Sau khi đã cấu hình public bridge, khi tạo máy ảo, bạn chỉ cần chọn chế độ "Bridge br0" là các máy ảo sẽ tự động được nhận các địa chỉ ip trùng với dải địa chỉ của card vật lí.</p>
<p>- Cơ chế cấp DHCP cho các máy ảo sẽ do Router bên ngoài đảm nhận, nhờ vậy nên các VM mới có dải địa chỉ ip trùng với card vật lí bên ngoài.</p>
<h3>3. Private Bridge</h3>
<p>- Chế độ này sẽ sử dụng một bridge riêng biệt để các VM giao tiếp với nhau mà không ảnh hưởng tới địa chỉ của KVM host.</p>
<p>- Ta có thể tạo ra private bridge bằng cách chỉnh sửa file <code>/etc/network/interfaces</code>. Tại đây, bạn sẽ không cần phải comment các cấu hình của card vật lý đồng thời không cần thêm tham số <code>bridge_ports</code> cho bridge.</p>
<p>- Bạn cũng có thể tạo ra private bridge bằng cách sử dụng virt-manager.</p>
<ul>
<li>Trong phần Edit, chọn Connection Details -> Virtual Networks</li>
<li>Chọn biểu tượng dấu `+`</li>
<li>Điền tên</li>
<li>Điền địa chỉ IP</li>
<li>Chọn `Isolated virtual network` và ấn Finish</li>
</ul>
<p>- Khi tạo máy ảo và kết nối tới private bridge, các máy ảo sẽ được cấp phát địa chỉ theo dải IP mà người dùng chọn. Chúng có thể giao tiếp với nhau những không đi ra được internet.</p>
<p>- Một máy ảo có thể được kết nối tới nhiều các private bridge, nhờ vậy nó có thể giao tiếp với nhau.</p>
<p>- Mô hình: </p>
<img src="https://camo.githubusercontent.com/5e010295b34d19aef71afb3b834afb8740223f80/687474703a2f2f692e696d6775722e636f6d2f4333466f6a7a522e706e67">
<p>- Nhờ việc gán nhiều card mạng mà VM ở dải 20.0.0.0 có thể ping tới máy ở dải 10.0.0.0</p>




