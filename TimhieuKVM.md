Tìm hiểu về KVM !
<h2> Mục lục </h2>  
<ul>
<h1>1. Khái niệm và vai trò. </h1>  
<h1>2. Cấu trúc của KVM. </h1>  
<h1>3.  Mô hình vận hành </h1>  
<h1>4.  Các tính năng của KVM</h1>  

</ul>
<ul>
<li>4.1. Security </li>
<li>4.2. Memory Management </li>
<li>4.3. Storage </li>
<li>4.4. Live migration   </li>
<li>4.5. Performance and scalability </li>

</ul>
<h1>5.  Tài liệu tham khảo</h1>  
<h3> 1. Khái niệm và vai trò.</h3>
<p>
•  KVM (Kernel-based virtual machine) là giải pháp ảo hóa cho hệ thống linux trên nền tảng phần cứng x86 có các module mở rộng hỗ trợ ảo hóa (Intel VT-x hoặc AMD-V).  
</p>
<p>
Về bản chất, KVM không thực sự là một hypervisor có chức năng giải lập phần cứng để chạy các máy ảo. Chính xác KVM chỉ là một module của kernel linux hỗ trợ cơ chế mapping các chỉ dẫn trên CPU ảo (của guest VM) sang chỉ dẫn trên CPU vật lý (của máy chủ chứa VM). Hoặc có thể hình dung KVM giống như một driver cho hypervisor để sử dụng được tính năng ảo hóa của các vi xử lý như Intel VT-x hay AMD-V, mục tiêu là tăng hiệu suất cho guest VM.   
</p>
<p>
•  KVM ban đầu được phát triển bởi Qumranet – một công ty nhỏ, sau đó được Redhat mua lại vào tháng 9 năm 2008. Ta có thể thấy KVM là thế hệ tiếp theo của công nghệ ảo hóa. KVM được sử dụng mặc định từ bản RHEL (Redhat Enterprise Linux) từ phiên bản 5.4 và phiên bản Redhat Enterprise Virtualization dành cho Server. 
</p>
<p>  
• Qumranet phát hành mã của KVM cho cộng đồng mã nguồn mở. Hiện nay, các công ty nổi tiếng như IBM, Intel và ADM cũng đã cộng tác với dự án. Từ phiên bản 2.6.20, KVM trở thành một phần của hạt nhân Linux.  
Bảng so sánh KVM với VMWare:  
</p>
<img src="https://imgur.com/a/wyn6B">
![](https://imgur.com/a/wyn6B)
<h3> 2. KVM Stack </h3>
<img src="https://imgur.com/a/1Z8Cz">
![](https://imgur.com/a/1Z8Cz)
<p>Trên đây là KVM Stack bao gồm 4 tầng:
User-facing tools: Là các công cụ quản lý máy ảo hỗ trợ KVM. Các công cụ có giao diện đồ họa (như virt-manager) hoặc giao diện dòng lệnh như (virsh)</p>
<p>
Management layer: Lớp này là thư viện libvirt cung cấp API để các công cụ quản lý máy ảo hoặc các hypervisor tương tác với KVM thực hiện các thao tác quản lý tài nguyên ảo hóa, vì tự thân KVM không hề có khả năng giả lập và quản lý tài nguyên như vậy.</p>
<p>
Virtual machine: Chính là các máy ảo người dùng tạo ra. Thông thường, nếu không sử dụng các công cụ như virsh hay virt-manager, KVM sẽ sử được sử dụng phối hợp với một hypervisor khác điển hình là QEMU.
Kernel support: Chính là KVM, cung cấp một module làm hạt nhân cho hạ tầng ảo hóa (kvm.ko) và một module kernel đặc biệt hỗ trợ các vi xử lý VT-x hoặc AMD-V (kvm-intel.ko hoặc kvm-amd.ko)
</p>
<p>Trong cấu trúc tổng quan của KVM bao gồm 3 thành phần chính:</p>

**<p>KVM kernel module:</p>**

<p>- Là một phần trong dòng chính của Linux kernel.</p>
<p>- Cung cấp giao diện chung cho Intel VMX và AMD SVM (thành phần hỗ trợ ảo hóa phần cứng)</p>
<p>- Chứa những mô phỏng cho các instructions và CPU modes không được support bởi Intel VMX và AMD SVM</p>
<p>- Quemu - kvm: là chương trình dòng lệnh để tạo các máy ảo, thường được vận chuyển dưới dạng các package "kvm" hoặc "qemu-kvm". </p>

<p>Có 3 chức năng chính:</p>
<p>- Thiết lập VM và các thiết bị ra vào (input/output)</p>
<p>- Thực thi mã khách thông qua KVM kernel module</p>
<p>- Mô phỏng các thiết bị ra vào (I/O) và di chuyển các guest từ host này sang host khác</p>
Libvirt management stack:
<p>- Cung cấp API để các tool như virsh có thể giao tiếp và quản lí các VM</p>
<p>- Cung cấp chế độ quản lí từ xa an toàn</p>
<p><li>KVM - QEMU </li></p>
<p>
Hệ thống ảo hóa KVM hay đi liền với QEMU. Về mặt bản chất, QEMU đã là một hypervisor hoàn chỉnh và là hypervisor loại 2. QEMU có khả năng giả lập tài nguyên phần cứng, trong đó bao gồm một CPU ảo. Các chỉ dẫn của hệ điều hành tác động lên CPU ảo này sẽ được QEMU chuyển đổi thành chỉ dẫn lên CPU vật lý nhờ một translator là TCG(Tiny Core Generator). Các hypervisor loại 2 khác như VMWare cũng có các bộ chuyển đổi tương tự, và bản thân các bộ dịch này hiệu suất không lớn. </p>
<p>
Do KVM hỗ trợ ánh xạ CPU vật lý sang CPU ảo, cung cấp khả năng tăng tốc phần cứng cho máy ảo và hiệu suất của nó nên QEMU sử dụng KVM làm accelerator tận dụng tính năng này của KVM thay vì sử dụng TCG.
</p>
<h3>3. Mô hình vận hành </h3>
<li>Hình dưới đây mô tả mô hình vận hành của KVM. Đây là một vòng lặp của các hành động diễn ra để vận hành các máy ảo. Những hành động này được phân cách bằng 3 phương thức chúng ta đã đề cập trước đó: user-mode, kernel-mode, guest-mode.</li>
<img src="https://imgur.com/a/HAfzw">
![](https://imgur.com/a/HAfzw)
<p>Như ta thấy: </p>
<li>User-mode: Các modul KVM gọi đến sử dụng ioclt() để thực thi mã khách cho đến khi hoạt động I/O khởi xướng bởi guest hoặc một sự kiện nào đó bên ngoài xảy ra. Sự kiện này có thể là sự xuất hiện của một gói tin mạng, cũng có thể là trả lời của một gói tin mạng được gửi bởi các may chủ trước đó. Những sự kiện như vậy được biểu diễn như là tín hiệu dẫn đến sự gián đoạn của thực thi mã khách.
</li>
<li>Kernel-mode: Kernel làm phần cứng thực thi các mã khách tự nhiên. Nếu bộ xử lý thoát khỏi guest do cấp phát bộ nhớ hay I/O hoạt động, kernel thực hiện các nhiệm vụ cần thiết và tiếp tục luồng thực hiện, Nếu các sự kiện bên ngoài như tín hiệu hoặc I/O hoạt động khởi xướng bởi các guest tồn tại, nó thoát tới user-mode.
</li>
<li>
Guest-mode: Đây là cấp độ phần cứng, nơi mà các lệnh mở rộng thiết lập của một CPU có khả năng ảo hóa được sử dụng để thực thi mã nguồn gốc, cho đến khi một lệnh được gọi như vậy cần sự hỗ trợ của KVM, một lỗi hoặc một gián đoạn từ bên ngoài.</li>

<li>
Khi một máy ảo chạy, có rất nhiều chuyển đổi giữa các chế độ. Từ kernel-mode tới guest-mode và ngược lại rất nhanh, bởi vì chỉ có mã nguồn gốc được thực hiện trên phần cứng cơ bản. Khi I/O hoạt động diễn ra các luồng thực thi tới user-mode, rất nhiều thiết bị ảo I/O được tạo ra, do vậy rất nhiều I/O thoát ra và chuyển sang chế độ user-mode chờ. Hãy tưởng tượng mô phỏng một đĩa cứng và 1 guest đang đọc các block từ nó. Sau đó QEMU mô phỏng các hoạt động bằng cách giả lập các hoạt động bằng các mô phỏng hành vi của các ổ đĩa cứng và bộ điều khiển nó được kết nối. Để thực hiện các hoạt động đọc, nó đọc các khối tương ứng từ một tập tin lớp và trả về dữ liệu cho guest. Vì vậy, user-mode giả lập I/O có xu hướng xuất hiện một nút cổ chai làm chậm việc thực hiện máy ảo.</li>
<p> Cơ chế hoạt động </p>
<li>Để các máy ảo giao tiếp được với nhau, KVM sử dụng Linux Bridge và OpenVSwitch, đây là 2 phần mềm cung cấp các giải pháp ảo hóa network. Trong bài này, tôi sẽ sử dụng Linux Bridge.</li>
<li>Linux bridge là một phần mềm đươc tích hợp vào trong nhân Linux để giải quyết vấn đề ảo hóa phần network trong các máy vật lý. Về mặt logic Linux bridge sẽ tạo ra một con switch ảo để cho các VM kết nối được vào và có thể nói chuyện được với nhau cũng như sử dụng để ra mạng ngoài.</li>
<li>Cấu trúc của Linux Bridge khi kết hợp với KVM - QEMU.</li>
![](https://imgur.com/a/UK8j1)
<ul><p>Ở đây: </p>

<li> Bridge: tương đương với switch layer 2 </li>
<li> Port: tương đương với port của switch thật </li>
<li> Tap (tap interface): có thể hiểu là giao diện mạng để các VM kết nối với bridge cho linux bridge tạo ra </li>
<li> fd (forward data): chuyển tiếp dữ liệu từ máy ảo tới bridge </li>
</ul>
<p>Các tính năng chính: </p>

<li>STP: Spanning Tree Protocol - giao thức chống lặp gói tin trong mạng
VLAN: chia switch (do linux bridge tạo ra) thành các mạng LAN ảo, cô lập traffic giữa các VM trên các VLAN khác nhau của cùng một switch.</li>
<li>FDB (forwarding database): chuyển tiếp các gói tin theo database để nâng cao hiệu năng switch. Database lưu các địa chỉ MAC mà nó học được. Khi gói tin Ethernet đến, bridge sẽ tìm kiếm trong database có chứa MAC address không. Nếu không, nó sẽ gửi gói tin đến tất cả các cổng.</li>

<h3> 4. Các tính năng của KVM </h3>
<li> 4.1. Security</li>
<img src="https://imgur.com/a/yxeIq">
![](https://imgur.com/a/yxeIq)
<p>Trong kiến trúc KVM, máy ảo được xem như các tiến trình Linux thông thường, nhờ đó nó tận dụng được mô hình bảo mật của hệ thống Linux như SELinux, cung cấp khả năng cô lập và kiểm soát tài nguyên. 
Bên cạnh đó còn có SVirt project - dự án cung cấp giải pháp bảo mật MAC (Mandatory Access Control - Kiểm soát truy cập bắt buộc) tích hợp với hệ thống ảo hóa sử dụng SELinux để cung cấp một cơ sở hạ tầng cho phép người quản trị định nghĩa nên các chính sách để cô lập các máy ảo. Nghĩa là SVirt sẽ đảm bảo rằng các tài nguyên của máy ảo không thể bị truy cập bởi bất kì các tiến trình nào khác; việc này cũng có thể thay đổi bởi người quản trị hệ thống để đặt ra quyền hạn đặc biệt, nhóm các máy ảo với nhau chia sẻ chung tài nguyên.</p>
<li>4.2. Memory Management </li>
<p>
KVM thừa kế tính năng quản lý bộ nhớ mạnh mẽ của Linux. Vùng nhớ của máy ảo được lưu trữ trên cùng một vùng nhớ dành cho các tiến trình Linux khác và có thể swap. KVM hỗ trợ NUMA (Non-Uniform Memory Access - bộ nhớ thiết kế cho hệ thống đa xử lý) cho phép tận dụng hiệu quả vùng nhớ kích thước lớn. 
KVM hỗ trợ các tính năng ảo của mới nhất từ các nhà cung cấp CPU như EPT (Extended Page Table) của Microsoft, Rapid Virtualization Indexing (RVI) của AMD để giảm thiểu mức độ sử dụng CPU và cho thông lượng cao hơn. 
KVM cũng hỗ trợ tính năng Memory page sharing bằng cách sử dụng tính năng của kernel là Kernel Same-page Merging (KSM).</p>
<li>4.3. Storage</li>
<p>KVM có khả năng sử dụng bất kỳ giải pháp lưu trữ nào hỗ trợ bởi Linux để lưu trữ các Images của các máy ảo, bao gồm các ổ cục bộ như IDE, SCSI và SATA, Network Attached Storage (NAS) bao gồm NFS và SAMBA/CIFS, hoặc SAN thông qua các giao thức iSCSI và Fibre Channel. 
KVM tận dụng được các hệ thống lưu trữ tin cậy từ các nhà cung cấp hàng đầu trong lĩnh vực Storage. 
KVM cũng hỗ trợ các images của các máy ảo trên hệ thống tệp tin chia sẻ như GFS2 cho phép các images có thể được chia sẻ giữa nhiều host hoặc chia sẻ chung giữa các ổ logic.</p>
<li>4.4. Live migration</li>
<p>KVM hỗ trợ live migration cung cấp khả năng di chuyển ác máy ảo đang chạy giữa các host vật lý mà không làm gián đoạn dịch vụ. Khả năng live migration là trong suốt với người dùng, các máy ảo vẫn duy trì trạng thái bật, kết nối mạng vẫn đảm bảo và các ứng dụng của người dùng vẫn tiếp tục duy trì trong khi máy ảo được đưa sang một host vật lý mới. KVM cũng cho phép lưu lại trạng thái hiện tại của máy ảo để cho phép lưu trữ và khôi phục trạng thái đó vào lần sử dụng tiếp theo.</p>
<li>4.5. Performance and scalability</li>
<p>KVM kế thừa hiệu năng và khả năng mở rộng của Linux, hỗ trợ máy ảo với 16 CPUs ảo, 256GB RAM và hệ thống máy host lên tới 256 cores và trên 1TB RAM.</p>

<p> [1] - http://www.ibm.com/developerworks/cloud/library/cl-hypervisorcompare-kvm/ </p>
<p> [2] - https://www.ibm.com/developerworks/library/l-using-kvm/ </p>
<p> [3] - https://manthang.wordpress.com/2014/06/18/kvm-qemu-do-you-know-the-connection-between-them/ </p>
