<h3> Tìm hiểu về iptables
</p>
Nội dung 
</h3>
<ul> 
<li>
<a href="#gioi_thieu"> 1. Giới thiệu </a>
</li> 
<li>
<a href="#bat_dau_tim_hieu"> 2. Bắt đầu tìm hiểu </a>
</li> 
<li>
<a href="#viet_mot_quy_tac_don_gian"> 3. Viết một bộ quy tắc đơn giản </a>
</li> 
<li>
<a href="#giao_dien"> 4. Giao diện </a>
</li> 
<li>
<a href="#cac_dia_chi_ip"> 5. Các địa chỉ IP </a>
</li> 
<li>
<a href="#port_va_protocols"> 6. Port và Protocols </a>
</li> 
<li>
<a href="#putting_it_all_together"> 7. Putting It All Together </a>
</li> 
<li>
<a href="#tom_tat"> 8. Tóm tắt </a>
</li>
 <li>
<a href="#lien_ket"> 9. Liên kết </a>
</li>
</ul>
<h4> 1. Giới thiệu </h4>
<p>CentOS có một tường lửa cực mạnh được xây dựng trong, thường được gọi là iptables, nhưng chính xác hơn là iptables / netfilter. Iptables là mô đun userpace, bit mà bạn, người dùng, tương tác với tại dòng lệnh để nhập các quy tắc tường lửa vào các bảng được xác định trước. Netfilter là một mô-đun hạt nhân, được tích hợp trong hạt nhân, thực sự lọc. Có nhiều giao diện người dùng GUI cho iptables, cho phép người dùng thêm hoặc xác định các quy tắc dựa trên một điểm và một giao diện người dùng, nhưng thường thiếu sự linh hoạt trong việc sử dụng giao diện dòng lệnh và giới hạn sự hiểu biết của người dùng về những gì thực sự xảy ra. Chúng ta sẽ học giao diện dòng lệnh của iptables.</p>
<p>Trước khi chúng ta có thể nắm bắt được với iptables, chúng ta cần phải có ít nhất một sự hiểu biết cơ bản về cách nó hoạt động. Iptables sử dụng khái niệm địa chỉ IP, các giao thức (tcp, udp, icmp) và các cổng. Chúng ta không cần phải là những chuyên gia trong lĩnh vực này để bắt đầu (vì chúng ta có thể tìm kiếm bất kỳ thông tin nào chúng ta cần), nhưng giúp bạn có được sự hiểu biết chung.</p>
<p>Iptables đặt các quy tắc vào các chuỗi được xác định trước (INPUT, OUTPUT và FORWARD) được kiểm tra đối với bất kỳ lưu lượng mạng nào (các gói IP) có liên quan đến các chuỗi đó và quyết định được làm gì với mỗi gói tin dựa trên kết quả của các quy tắc đó, hoặc bỏ gói tin. Các hành động này được gọi là các mục tiêu, trong đó hai mục tiêu được xác định trước phổ biến nhất là DROP để thả một gói hoặc ACCEPT để chấp nhận một gói tin.</p>
<li>
Chains
</li>
<p>Đây là 3 chuỗi được xác định trước trong bảng lọc mà chúng ta có thể thêm các quy tắc để xử lý các gói tin IP đi qua các chuỗi đó. Các chuỗi này là:</p>
<ul>
<li>INPUT - Tất cả các gói tin dành cho máy chủ.</li>
<li>OUTPUT - Tất cả các gói tin có nguồn gốc từ máy chủ.</li>
<li>FORWARD - Tất cả các gói tin không dành cho cũng không bắt nguồn từ máy tính chủ, nhưng đi qua (chuyển bởi) máy tính chủ. Chuỗi này được sử dụng nếu bạn đang sử dụng máy tính như một bộ định tuyến.</li>
</ul>
<p>Đối với hầu hết các phần, chúng ta sẽ giải quyết chuỗi INPUT để lọc các gói dữ liệu nhập vào máy tính của chúng ta - nghĩa là giữ các kẻ xấu.</p>
<p>Các quy tắc được thêm vào một danh sách cho mỗi chuỗi. Một gói được kiểm tra đối với mỗi quy tắc lần lượt, bắt đầu từ đầu, và nếu nó phù hợp với quy tắc đó thì hành động được thực hiện như chấp nhận (ACCEPT) hoặc thả (DROP) gói. Khi một quy tắc đã được kết hợp và một hành động được thực hiện, gói tin sẽ được xử lý theo kết quả của quy tắc đó và không được xử lý bởi các quy tắc khác trong chuỗi. Nếu một gói tin đi qua tất cả các quy tắc trong chuỗi và đạt đến đáy mà không khớp với bất kỳ quy tắc nào, thì hành động mặc định cho chuỗi đó được thực hiện. Đây được gọi là chính sách mặc định và có thể được đặt thành ACCEPT hoặc DROP gói.</p>
<h4>
 Khái niệm chính sách mặc định trong chuỗi tạo ra hai khả năng cơ bản mà chúng ta phải xem xét trước khi quyết định cách chúng ta sắp xếp tường lửa của chúng ta.
</h4>
<p>1. Chúng ta có thể thiết lập một chính sách mặc định để DROP tất cả các gói tin và sau đó thêm các quy tắc để đặc biệt cho phép (ACCEPT) các gói tin có thể được từ các địa chỉ IP đáng tin cậy, hoặc cho một số cổng mà chúng ta có các dịch vụ chạy như bittorrent, FTP server, Web Server , Máy chủ tệp Samba ...</p>
<p>Hay cách khác,</p>
<p>2. Chúng ta có thể thiết lập chính sách mặc định để ACCEPT tất cả các gói dữ liệu và sau đó thêm các quy tắc để chặn các gói đặc biệt (DROP) có thể là từ các địa chỉ IP hoặc các vùng địa chỉ IP gây phiền toái đặc biệt hoặc cho một số cổng mà chúng ta có các dịch vụ riêng hoặc không có các dịch vụ đang chạy.</p>
<p>Nói chung, tùy chọn 1 ở trên được sử dụng cho chuỗi INPUT nơi chúng tôi muốn kiểm soát những gì được phép truy cập vào máy tính của chúng tôi và tùy chọn 2 sẽ được sử dụng cho chuỗi OUTPUT, nơi chúng ta thường tin tưởng vào lưu lượng truy cập đang để lại (có nguồn gốc từ) máy của chúng tôi.</p>
<h4> 2. Bắt đầu tìm hiểu </h4>
<p>Làm việc với iptables từ dòng lệnh đòi hỏi quyền root, vì vậy bạn cần phải trở thành root cho hầu hết mọi thứ chúng ta sẽ làm.</p>
<pre>QUAN TRỌNG: Chúng ta sẽ tắt iptables và đặt lại các quy tắc tường lửa của bạn, vì vậy nếu bạn dựa vào tường lửa Linux làm tuyến phòng thủ chính, bạn nên biết về điều này.</pre>
<p>Iptables nên được cài đặt mặc định trên tất cả các CentOS 5.x và 6.x cài đặt. Bạn có thể kiểm tra để xem nếu iptables được cài đặt trên hệ thống của bạn bằng cách:</p>
<pre>
$ rpm-q iptables
iptables-1.4.7-5.1.el6_2.x86_64
</pre>
<p>Và để xem nếu iptables thực sự chạy, chúng ta có thể kiểm tra rằng các mô-đun iptables được nạp và sử dụng -L để kiểm tra các quy tắc hiện đang được nạp:</p>
<pre>
# lsmod | grep ip_tables
ip_tables 29288 1 iptable_filter
x_tables 29192 6 ip6t_REJECT, ip6_tables, ipt_REJECT, xt_state, xt_tcpudp, ip_tables
</pre>
<pre>
# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            state RELATED,ESTABLISHED 
ACCEPT     icmp --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     tcp  --  anywhere             anywhere            state NEW tcp dpt:ssh 
REJECT     all  --  anywhere             anywhere            reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  anywhere             anywhere            reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination        
</pre>

<p>Ở trên, chúng ta sẽ thấy tập hợp các quy tắc mặc định trên một hệ thống CentOS 6. Lưu ý rằng dịch vụ SSH được cho phép theo mặc định.</p>
<p>Nếu iptables không chạy, bạn có thể kích hoạt nó bằng cách chạy:</p>

<code>
# system-config-securitylevel
</code>
<h4> 3. Viết một bộ quy tắc đơn giản  </h4>
<code>
QUAN TRỌNG: Tại thời điểm này chúng ta sẽ xóa bộ quy tắc mặc định. Nếu bạn đang kết nối từ xa đến một máy chủ thông qua SSH cho hướng dẫn này sau đó có một khả năng rất thực tế mà bạn có thể khóa mình ra khỏi máy tính của bạn. Bạn phải đặt chính sách đầu vào mặc định để chấp nhận trước khi làm sạch các quy tắc hiện tại, và sau đó thêm một quy tắc vào đầu để rõ ràng cho phép bạn truy cập để ngăn chặn chống lại chính mình.
</code>
<p>Chúng ta sẽ sử dụng cách tiếp cận dựa trên ví dụ để kiểm tra các lệnh iptables khác nhau. Trong ví dụ đầu tiên, chúng ta sẽ tạo ra một bộ quy tắc rất đơn giản để thiết lập tường lửa SPI (Stateful Packet Inspection) cho phép tất cả các kết nối đi ra nhưng chặn tất cả các kết nối không mong muốn:</p>
<pre># iptables -P INPUT ACCEPT
# iptables -F
# iptables -A INPUT -i lo -j ACCEPT
# iptables -A INPUT -m state --state ESTABLISHED, RELATED -j ACCEPT
# iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# iptables -P INPUT DROP
# iptables -P FORWARD DROP
# iptables -P OUTPUT ACCEPT
# iptables -L -v</pre>

<p> đầu ra sau đây:</p>
<pre>Chain INPUT (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     all  --  lo     any     anywhere             anywhere
    0     0 ACCEPT     all  --  any    any     anywhere             anywhere            state RELATED,ESTABLISHED
    0     0 ACCEPT     tcp  --  any    any     anywhere             anywhere            tcp dpt:ssh
Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
</pre>

<p>Bây giờ hãy nhìn vào mỗi trong 8 lệnh ở trên và hiểu chính xác những gì chúng ta vừa làm:</p>
<p>1. iptables -P INPUT ACCEPT Nếu kết nối từ xa trước hết chúng ta phải tạm thời thiết lập chính sách mặc định cho chuỗi INPUT để ACCEPT nếu không thì khi chúng ta thực hiện các quy tắc hiện tại, chúng ta sẽ bị khóa khỏi máy chủ của chúng tôi.</p>
<p>2. iptables -F Chúng tôi sử dụng -F để chuyển đổi tất cả các quy tắc hiện có để chúng ta bắt đầu với một trạng thái sạch sẽ để thêm các quy tắc mới.</p>
<p>3. iptables -A INPUT -i lo -j ACCEPT Bây giờ là lúc bắt đầu thêm một số quy tắc. Chúng ta sử dụng -A để chuyển sang thêm (hoặc thêm) một quy tắc vào một chuỗi cụ thể, chuỗi INPUT trong trường hợp này. Sau đó, chúng ta sử dụng -i chuyển đổi (cho giao diện) để xác định các gói phù hợp hoặc định cho giao diện lo (localhost, 127.0.0.1) và cuối cùng là -j (nhảy) tới hành động đích cho các gói phù hợp với quy tắc - trong trường hợp ACCEPT. Vì vậy, quy tắc này sẽ cho phép tất cả các gói tin đến cho giao diện localhost được chấp nhận. Điều này thường được yêu cầu như nhiều ứng dụng phần mềm mong đợi để có thể giao tiếp với bộ điều hợp localhost.</p>
<p>4. iptables -A INPUT -m state --state ESTABLISHED, RELATED -j ACCEPT Đây là nguyên tắc mà hầu hết công việc, và một lần nữa chúng ta thêm (-A) vào chuỗi INPUT. Ở đây chúng ta đang sử dụng -m switch để load một module (state). Mô-đun nhà nước có thể kiểm tra trạng thái của một gói tin và xác định nó là MỚI, THÀNH LẬP hay LIÊN QUAN. NEW đề cập đến các gói tin đến là các kết nối mới mà không được khởi tạo bởi hệ thống máy chủ lưu trữ. ESTABLISHED và RELATED đề cập đến các gói tin gửi đến là một phần của một kết nối đã được thiết lập hoặc có liên quan đến và kết nối đã được thiết lập.</p>
<p>5. iptables -A INPUT -p tcp -dport 22 -j ACCEPT Ở đây chúng ta thêm một quy tắc cho phép kết nối SSH qua tcp port 22. Đây là cách để ngăn chặn tình cờ lockouts khi làm việc trên các hệ thống từ xa qua kết nối SSH. Chúng ta sẽ giải thích chi tiết hơn về quy tắc này sau.</p>
<p>6. iptables -P INPUT DROP Các -P chuyển đặt chính sách mặc định trên chuỗi quy định. Vì vậy bây giờ chúng ta có thể thiết lập chính sách mặc định trên chuỗi INPUT để DROP. Điều này có nghĩa là nếu một gói đến không khớp với một trong các quy tắc sau đây nó sẽ bị bỏ. Nếu chúng tôi kết nối từ xa thông qua SSH và không thêm quy tắc trên, chúng ta sẽ chỉ khóa chúng tôi ra khỏi hệ thống vào thời điểm này.</p>
<p>7. iptables -P HƯỚNG DROP Tương tự như vậy, ở đây chúng tôi đã thiết lập chính sách mặc định trên chuỗi mong muốn được thả như chúng ta không sử dụng máy tính của chúng tôi như một router vì vậy không nên có bất kỳ gói tin đi qua máy tính của chúng tôi.</p>
<p>8. iptables -P OUTPUT ACCEPT và cuối cùng, chúng tôi đã thiết lập chính sách mặc định trên chuỗi OUTPUT để CHẤP NHẬN như chúng ta muốn cho phép tất cả lưu lượng đi (như chúng ta tin tưởng người dùng của chúng ta).</p>
<p>9. iptables -L -v Cuối cùng, chúng ta có thể liệt kê (-L) các quy tắc mà chúng ta vừa bổ sung để kiểm tra chúng đã được nạp chính xác hay không.</p>

<p>Cuối cùng, điều cuối cùng chúng ta cần làm là lưu lại các quy tắc của chúng tôi để lần sau khi chúng ta khởi động lại máy tính của chúng tôi các quy tắc của chúng tôi được tự động nạp lại:
</p>
<code>
 # /sbin/service iptables save
</code>
<p>Điều này thực thi tập lệnh iptables init, chạy / sbin / iptables-save và ghi cấu hình iptables hiện tại vào / etc / sysconfig / iptables. Khi khởi động lại, tập lệnh init của iptables reapplies các quy tắc được lưu trong /etc/sysconfig/iptables bằng cách sử dụng lệnh /sbin/iptables-restore .</p>
<p>Rõ ràng gõ tất cả các lệnh này vào trình bao có thể trở nên nhàm chán, do đó, cách dễ nhất để làm việc với iptables là tạo một tập lệnh đơn giản để làm tất cả cho bạn. Các lệnh ở trên có thể được nhập vào trình soạn thảo văn bản ưa thích của bạn và được lưu dưới dạng tệp văn bản cảnh báo, ví dụ:</p>
<pre>
#!/bin/bash
#
# iptables example configuration script
#
# Flush all current rules from iptables
#
 iptables -F
#
# Allow SSH connections on tcp port 22
# This is essential when working on remote servers via SSH to prevent locking yourself out of the system
#
 iptables -A INPUT -p tcp --dport 22 -j ACCEPT
#
# Set default policies for INPUT, FORWARD and OUTPUT chains
#
 iptables -P INPUT DROP
 iptables -P FORWARD DROP
 iptables -P OUTPUT ACCEPT
#
# Set access for localhost
#
 iptables -A INPUT -i lo -j ACCEPT
#
# Accept packets belonging to established and related connections
#
 iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
#
# Save settings
#
 /sbin/service iptables save
#
# List rules
#
 iptables -L -v
</pre>
<p>Lưu ý: Chúng ta cũng có thể bình luận kịch bản của chúng ta để nhắc nhở chúng ta những gì đã làm.</p>
<p>Chạy lệnh:</p>
<pre># chmod + x myfirewall</pre>
<p> Bây giờ chúng ta có thể chỉ cần chỉnh sửa kịch bản của chúng ta và chạy nó từ trình bao bằng lệnh sau:</p>
<code># ./myfirewall</code>
<h4>4. Giao diện</h4>
<p>Trong ví dụ trước của chúng ta, chúng ta đã thấy cách chúng ta có thể chấp nhận tất cả các gói dữ liệu đến trên một giao diện cụ thể, trong trường hợp này là giao diện localhost:</p>
<pre>iptables -A INPUT -i lo -j ACCEPT</pre>
<p> Giả sử chúng ta có 2 giao diện riêng biệt, eth0 là kết nối nội bộ LAN và modem dialup ppp0 (hoặc có thể eth1 cho một nic) mà là kết nối internet bên ngoài của chúng tôi. Chúng ta có thể muốn cho phép tất cả các gói tin gửi đến trên mạng LAN nội bộ của chúng tôi nhưng vẫn lọc các gói dữ liệu đến trên kết nối internet bên ngoài của chúng tôi. Chúng ta có thể làm như sau:</p>
<code>iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i eth0 -j ACCEPT</code>
<p> Nhưng hãy cẩn thận - nếu chúng ta để cho phép tất cả các gói cho giao diện internet bên ngoài của chúng ta (ví dụ, modem quay số ppp0):</p>
<code>iptables -A INPUT -i ppp0 -j ACCEPT</code>
<p> Chúng ta đã có thể có hiệu quả chỉ cần vô hiệu tường lửa của chúng ta!</p>
<h4>5. Các địa chỉ IP</h4>
<p> Việc mở rộng toàn bộ giao diện cho các gói tin đến có thể không đủ hạn chế và bạn có thể muốn kiểm soát nhiều hơn về những gì để cho phép và những gì để từ chối. Cho phép giả sử chúng ta có một mạng máy tính nhỏ sử dụng mạng con riêng 192.168.0.x. Chúng ta có thể mở tường lửa của chúng ta đến các gói tin gửi đến từ một địa chỉ IP tin cậy duy nhất (ví dụ 192.168.0.4):</p>
<code># Accept packets from trusted IP addresses
 iptables -A INPUT -s 192.168.0.4 -j ACCEPT # change the IP address as appropriate</code>
<p> Breaking lệnh này xuống, trước tiên chúng ta nối (-A) một quy tắc vào chuỗi INPUT cho địa chỉ IP 192.168.0.4 (để nhận ACCEPT) tất cả các gói (cũng lưu ý chúng ta có thể sử dụng ký hiệu # để thêm ý kiến nội tuyến vào tài liệu kịch bản của chúng ta với bất cứ điều gì sau khi # bị bỏ qua và coi như là một nhận xét).</p>
<p> Rõ ràng nếu chúng ta muốn cho phép các gói tin đến từ một loạt các địa chỉ IP, chúng ta chỉ cần thêm một quy tắc cho mỗi địa chỉ IP đáng tin cậy và điều đó sẽ làm việc tốt. Nhưng nếu chúng ta có rất nhiều trong số họ, có thể dễ dàng hơn để thêm một loạt các địa chỉ IP trong một lần. Để làm điều này, chúng ta có thể sử dụng một netmask hoặc ký hiệu slash chuẩn để xác định một phạm vi địa chỉ IP. Ví dụ: nếu chúng ta muốn mở tường lửa cho tất cả các gói tin gửi đến từ phạm vi 192.168.0.x (ở đó x = 1 đến 254), chúng ta có thể sử dụng một trong hai phương pháp sau:</p>
<pre># Accept packets from trusted IP addresses
 iptables -A INPUT -s 192.168.0.0/24 -j ACCEPT  # using standard slash notation
 iptables -A INPUT -s 192.168.0.0/255.255.255.0 -j ACCEPT # using a subnet mask</pre>
<p> Cuối cùng, cũng như lọc đối với một địa chỉ IP duy nhất, chúng ta cũng có thể phù hợp với địa chỉ MAC cho thiết bị đã cho. Để làm điều này, chúng ta cần tải một mô-đun (mô đun mac) cho phép lọc chống lại các địa chỉ mac. Trước đó chúng ta đã thấy một ví dụ khác về việc sử dụng các mô-đun để mở rộng chức năng của iptables khi chúng ta sử dụng mô-đun nhà nước để khớp với các gói tin ESTABLISHED và RELATED. Ở đây chúng ta sử dụng mô đun mac để kiểm tra địa chỉ mac của nguồn gói tin ngoài địa chỉ IP của nó:
</p>
<pre># Accept packets from trusted IP addresses
 iptables -A INPUT -s 192.168.0.4 -m mac --mac-source 00:50:8D:FD:E6:32 -j ACCEPT</pre>
<p> Đầu tiên chúng ta sử dụng -m mac để tải mô đun mac và sau đó chúng ta sử dụng --mac-source để xác định địa chỉ mac của địa chỉ IP nguồn (192.168.0.4). Bạn sẽ cần phải tìm ra địa chỉ MAC của mỗi thiết bị Ethernet mà bạn muốn lọc. Chạy ifconfig (hoặc iwconfig cho các thiết bị không dây) như là root sẽ cung cấp cho bạn địa chỉ mac.</p>
<p> Điều này có thể hữu ích trong việc ngăn ngừa giả mạo địa chỉ IP nguồn vì nó sẽ cho phép bất kỳ gói nào thực sự có nguồn gốc từ 192.168.0.4 (có địa chỉ MAC 00: 50: 8D: FD: E6: 32) nhưng sẽ chặn mọi gói bị giả mạo đến từ địa chỉ đó. Lưu ý, lọc địa chỉ mac sẽ không hoạt động trên internet nhưng nó chắc chắn hoạt động tốt trên mạng LAN.</p>
<h4>6. Port và Protocols</h4>

<p> Ở trên chúng ta đã thấy làm thế nào chúng ta có thể thêm các quy tắc để tường lửa của chúng tôi để lọc với các gói tin phù hợp với một giao diện cụ thể hoặc một địa chỉ IP nguồn. Điều này cho phép truy cập đầy đủ thông qua tường lửa của chúng tôi đến một số nguồn đáng tin cậy (máy chủ). Bây giờ chúng ta sẽ xem xét làm thế nào chúng ta có thể lọc chống lại các giao thức và cổng để tinh chỉnh thêm những gói dữ liệu đến mà chúng ta cho phép và những gì chúng ta chặn.</p>
<p> Trước khi chúng ta có thể bắt đầu, chúng ta cần phải biết giao thức và số cổng mà một dịch vụ nhất định sử dụng. Đối với một ví dụ đơn giản, hãy nhìn vào bittorrent. Bittorrent sử dụng giao thức tcp trên cổng 6881, vì vậy chúng ta cần phải cho phép tất cả các gói tcp trên cổng đích (cổng mà chúng đến máy của chúng ta) 6881:</p>
<pre>
# Accept tcp packets on destination port 6881 (bittorrent)
 iptables -A INPUT -p tcp --dport 6881 -j ACCEPT
</pre>
<p>Ở đây chúng ta nối (-A) một quy tắc vào chuỗi INPUT cho các gói tin phù hợp với giao thức tcp (  -p tcp ) và nhập máy của chúng ta vào cổng đích 6881 ( --dport 6881 ).</p>
<p>Lưu ý: Để sử dụng các trận đấu như đích hoặc nguồn cổng ( --dport hoặc --sport ), bạn phải đầu tiên chỉ định giao thức (TCP, UDP, ICMP, tất cả).</p>
<p>Chúng ta cũng có thể mở rộng ở trên để bao gồm một dải cổng, ví dụ, cho phép tất cả các gói tin tcp trên dải từ 6881 đến 6890:
</p>
<p># Accept tcp packets on destination ports 6881-6890
 iptables -A INPUT -p tcp --dport 6881:6890 -j ACCEPT</p>
 <h4>7. Putting It All Together</h4>
<p>Bây giờ chúng ta đã thấy những điều cơ bản, chúng ta có thể bắt đầu kết hợp các quy tắc này.</p>
<p>Một dịch vụ UNIX / Linux phổ biến là dịch vụ SSH (secure shell) cho phép đăng nhập từ xa. Mặc định SSH sử dụng cổng 22 và một lần nữa sử dụng giao thức tcp. Vì vậy, nếu chúng ta muốn cho phép đăng nhập từ xa, chúng ta cần cho phép các kết nối tcp trên cổng 22:</p>
<pre># Accept tcp packets on destination port 22 (SSH)
 iptables -A INPUT -p tcp --dport 22 -j ACCEPT</pre>
<p>Điều này sẽ mở ra cổng 22 (SSH) cho tất cả các kết nối tcp đến, đây là một mối đe dọa tiềm ẩn về an ninh vì tin tặc có thể thử lực nứt trên các tài khoản có mật khẩu yếu. Tuy nhiên, nếu chúng ta biết địa chỉ IP của các máy từ xa đáng tin cậy sẽ được sử dụng để đăng nhập bằng SSH, chúng ta có thể hạn chế quyền truy cập vào chỉ các địa chỉ IP nguồn này. Ví dụ: nếu chúng ta chỉ muốn mở truy cập SSH trên LAN cá nhân (192.168.0.x), chúng tôi có thể giới hạn truy cập vào phạm vi địa chỉ IP nguồn này:</p>
<pre># Accept tcp packets on destination port 22 (SSH) from private LAN
 iptables -A INPUT -p tcp -s 192.168.0.0/24 --dport 22 -j ACCEPT
</pre>
<p>Sử dụng bộ lọc IP nguồn cho phép chúng tôi mở an toàn quyền truy cập SSH trên cổng 22 tới địa chỉ IP đáng tin cậy. Ví dụ, chúng ta có thể sử dụng phương pháp này để cho phép đăng nhập từ xa giữa công việc và máy chủ. Đối với tất cả các địa chỉ IP khác, cổng (và dịch vụ) sẽ đóng kín như thể dịch vụ đã bị vô hiệu hóa để các hacker sử dụng các phương pháp quét cổng có thể sẽ vượt qua chúng ta.</p>
<h4>8. Tóm tắt</h4>
<p>Chúng ta đã không nhận ra bề mặt của những gì có thể đạt được với iptables, nhưng hy vọng bài này đã cung cấp nền tảng tốt từ những điều cơ bản từ đó có thể xây dựng các bộ quy tắc phức tạp hơn.</p>
<h4>9. Liên kết</h4>
<p>1. http://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-fw.html</p>
<p>2. http://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-iptables.html</p>
<p>3. http://ip2location.com/free/visitor-blocker</p>


