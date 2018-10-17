<h3>1. Vai trò của project Cinder</h3>
<p>Cinder là dịch vụ Block storage trong Openstack.Nó được thiết kế để người sử dụng cuối có thể thực hiện việc lưu trữ bởi Nova,việc này được thực hiện bởi LVM hoặc các nền tảng lưu trữ khác.Cinder ảo hóa việc quản lý các thiết bị block storage và cung cấp cho người dùng cuối 1 API đáp ứng được nhu cầu tự phục vụ cũng như sử dụng các tài nguyên đó mà không cần biết quá nhiều kiến thức chuyên sâu.


<h3>2. Các thành phần của project Cinder </h3>
<img src="https://github.com/anhict/images/raw/master/11.png">
<p>Cinder-api: là một ứng dụng WSGI chấp nhận và xác nhận các yêu cầu REST (JSON hoặc XML) từ client và chuyển chúng tới các quy trình Cinder khác nếu thích hợp với AMQP</p>
<p>Cinder scheduler: Lên lịch và định tuyến các yêu cầu tới dịch vụ volume thích hợp. Tùy thuộc vào cách cấu hình có thể là dùng round-robin để định ra việc sẽ dùng volume service nào hoặc có thể phức tạp hơn bằng cách dùng Filter scheduler.Filter scheduler là mặc định và bật các bộ lọc như Capacity,Avaibility zone,Volume type và Capability.</p>
<p>Cinder-volume: Quản lí các thiết bị block storage,đặc biệt các các thiết bị back-end
<p>Cinder-backup: Cung cấp phương thức để backup một Block storage volume tới Openstack Oject storage(Swift)Khi một máy client yêu cầu sao lưu volume được tạo ra hoặc quản lý.</p>
<p>SQLDB: Cung cấp 1 phương tiện dùng để backup dữ liệu từ Switf/ceph ...</p>
<ul><li>Back-end storage device: Dịch vụ Block storage yêu cầu một vài kiểm của back-end storage mà dịch vụ có thể chạy trên đó.Mặc định là sử dụng LVM trên một local volume group tên là "Cinder-volume"</li>
<li>User và project: Cinder được dùng bởi người dùng hoặc khách hàng khác nhau(project trong 1 share system),sử dụng chỉ định truy cập dựa vào role(role based access).Các role kiểm soát các hành động mà người dùng được phép thực hiện.Trong cấu hình mặc định,phần lớn các hành động không yêu cầu 1 role cụ thể nhưng sysadmin có thể cấu hình trong file policy.json để quản lý các rule.Mộ truy cập của người dùng có thể bị hạn chế bởi project,nhưng username và pass được gán chỉ định cho mỗi user.Keypair cho phép truy cập tới 1 volume được mở cho mỗi user,nhưng hạn ngạch để kiểm soát sự sử dụng tài nguyên trên các tài nguyên phần cứng có sẵn là mỗi project.</li></ul>
<ul><li>Volume, snapshot và backup</li>
 <ul>
   <li>Volume: Các tài nguyên block storage được phân phối để có thể gán vào máy ảo như một ổ lưu trữ thứ 2 hoặc có thể dùng như là vùng lưu trữ cho root để boot vào máy ảo.Volume là các thiết bị bloc storage R/W bền vững thường được dùng để gán vào compute node thông qua iSCSI.</li> 
   <li>Snapshot: Một bản copy trong một thời điểm nhất định của volume snapshot có thể được tạo từ một volume mà mới được dùng gần đây trong trạng thái sẵn sàng.Snapshot có thể được dùng để tạo một volume mới thông qua việc tạo từ snapshot.</li>
   <li>Backup: Một bản copy lưu trữ của 1 volume thông thường được lưu ở swift.</li>
 </ul> 
</ul>
<h3>3. Luồng làm việc của Cinder</h3>
<h4>3.1 Workflow của Cinder khi tạo mới Volume </h4>
<img src="https://github.com/anhict/images/raw/master/12.png">
<p>Hình bên trên mô tả quy trình tạo Volume , tiếp theo chúng ta cùng đến với quy trình tạo ra volume mới của Cinder :</p>
<img src="https://github.com/anhict/images/raw/master/14.png">

<p>1.	Client yêu cầu tạo ra Volume thông qua việc gọi REST API (Client cũng có thể sử dụng tiện ích CLI của python-client)</p>

<p>2.	Cinder-api : Quá trình xác nhận hợp lệ yêu cầu thông tin người dùng , một khi được xác nhận một message được gửi lên hàng chờ AMQP để xử lý.</p>

<p>3.	Cinder-volume thực hiện quá trình đưa message ra khỏi hàng đợi , gửi thông báo tới cinder-scheduler để báo cáo xác định backend cung cấp volume.</p>

<p>4.	Cinder-scheduler thực hiện quá trình báo cáo sẽ đưa thông báo ra khỏi hàng đợi , tạo danh sách các ứng viên dựa trên trạng thái hiện tại và yêu cầu tạo volume theo tiêu chí (kích thước, vùng sẵn có, loại volume (bao gồm cả thông số kỹ thuật bổ sung)).</p>

<p>5.	Cinder-volume thực hiện quá trình đọc message phản hồi từ cinder-scheduler từ hàng đợi. Lặp lại qua các danh sách ứng viên bằng các gọi backend driver cho đến khi thành công.</p>

<p>6.	NetApp Cinder tạo ra volume được yêu cầu thông qua tương tác với hệ thống lưu trữ con (phụ thuộc vào cấu hình và giao thức).</p>

<p>7.	Cinder-volume thực hiện quá trình thu thập dữ liệu và metadata volume và thông tin kết nối để trả lại thông báo đến AMQP.</p>

<p>8.	Cinder-api thực hiện quá trình đọc message phản hồi từ hàng đợi và đáp ứng tới client.</p>

<p>9.	Client nhận được thông tin bao gồm trạng thái của yêu cầu tạo, Volume UUID, .... </p>
<h4>3.2 Workflow của Cinder khi attact Volume</h3>
<img src="https://github.com/anhict/images/raw/master/15.png">
<p>1.	Client yêu cầu attach volume thông qua Nova REST API (Client có thể sử dụng tiện ích CLI của python-novaclient)</p>

<p>2.	Nova-api thực hiện quá trình xác nhận yêu cầu và thông tin người dùng. Một khi đã được xác thực, gọi API Cinder để có được thông tin kết nối cho volume được xác định.</p>

<p>3.	Cinder-api thực hiện quá trình xác nhận yêu cầu hợp lệ và thông tin người dùng hợp lệ . Một khi được xác nhận , một message sẽ được gửi đến người quản lý volume thông qua AMQP.</p>

<p>4.	Cinder-volume tiến hành đọc message từ hàng đợi , gọi Cinder driver tương ứng với volume được gắn vào.</p>

<p>5.	NetApp Cinder driver chuẩn bị Cinder Volume chuẩn bị cho việc attach (các bước cụ thể phụ thuộc vào giao thức lưu trữ được sử dụng).</p>

<p>6.	Cinder-volume thưc hiện gửi thông tin phản hồi đến cinder-api thông qua hàng đợi AMQP.</p>

<p>7.	Cinder-api thực hiện quá trình đọc message phản hồi từ cinder-volume từ hàng đợi; Truyền thông tin kết nối đến RESTful phản hồi gọi tới NOVA.</p>

<p>8.	Nova tạo ra kết nối với bộ lưu trữ thông tin được trả về Cinder.</p>

<p>9.	Nova truyền volume device/file tới hypervisor , sau đó gắn volume device/file vào máy ảo client như một block device thực thế hoặc ảo hóa (phụ thuộc vào giao thức lưu trữ).</p>
<ul><li>Có 2 loại volume:</li><ul><li></li>-	Bootable<li>-	Non-bootable</li></ul></ul>
