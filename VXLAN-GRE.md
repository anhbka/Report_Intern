<p><strong>Generic Routing Encapsulation (GRE)</strong>: là một giao thức được phát triển bởi Cisco, cho phép đóng gói nhiều giao thức ở tầng Network layer trong kết nối point-to-point.</p>
<p>Một tunnel GRE được sử dụng khi cần gửi gói tin từ mạng này qua mạng khác hoặc mạng không an toàn. Với GRE, một virtual tunnel được tạo ra giữa hai router đầu cuối và gói tin được đóng gói gửi qua virtual tunnel.</p> 
<p>Một điều quan trọng là các gói tin gửi qua GRE chưa được mã hóa vì GRE không mã hóa mà chỉ đóng một hearder GRE. Nếu cần mã hóa, thì phải cấu hình thêm IPsec.
Quy trình đóng gói khi qua GRE không mã hóa :</p>
<img src="https://github.com/anhict/images/blob/master/cisco-routers-gre-2.png">
<p>Sự khác biệt cơ bản giữa hai loại này là GRE IPsec tunnel cho phép gói tin multicast gửi qua còn IPsec VPN thì không hỗ trợ gói tin này. Trong một mạng lớn khi chúng ta cần cầu hình OSPF, EIGRP....GRE tunnel là sự lựa chọn tốt nhất. Và phần cấu hình GRE cũng đơn giản hơn nhiều so với IPsec VPN nên hầu hết sẽ ưu tiên GRE tunnel hơn là IPsec VPN.</p>
