<h3>Network Filesystem</h3>
<p>NFS (the Network File System) là một giao thức được dùng cho việc chia sẻ data qua physical systems. Người quản trị gắn các thư mục của người dùng từ xa trên một máy chủ để cho phép họ truy cập vào cùng một tệp và cấu hình.</p>
<p>Hoạt động theo cơ chế client-server</p>
<p>Một số vấn đề với NFS</p>
<ul>
<li>Không bảo mật, mã hóa dữ liệu</li>
<li>Hiệu suất hoạt động trung bình ở mức khá, nhưng không ổn định</li>
<li>Dữ liệu phân tán có thể bị phá vỡ nếu có nhiều phiên sử dụng đồng thời</li>
</ul>
<p>File <code>/etc/export</code> chứa các đường dẫn thư mục và quyền hạn mà một host muốn chia sẻ dữ liệu với host khác qua NSF.</p>
<p>Các máy chủ có quyền hạn sau:</p>
<ul>
<li><code>rw</code>: Đọc và ghi</li>
<li><code>ro</code>: Chỉ được đọc</li>
<li><code>noacess</code>: Cấm truy cập vào các thư mục con của thư mục đc chia sẻ</li>
</ul>
<p>Ví dụ bạn muốn chia sẻ thư mục <code>/share</code> cho các máy có địa chỉ trong 192.168.1.1/28 có quyền đọc ghi thì thêm vào nội dung file dòng sau:</p>
<pre><code>/Share 192.168.1.1/28(rw)
</code></pre>
<h3>Cài đặt trên Ubuntu</h3>
<h4>Installation</h4>
<p>Cài đặt nfs trên server:</p>
<pre><code>sudo apt-get install nfs-kernel-server
</code></pre>
<p>Trên client sẽ cài một gói nfs-common cung cấp chức năng nfs mà không bao gồm các thành phần server không cần thiết.</p>
<pre><code>sudo apt-get install nfs-common
</code></pre>
<p>Khởi động nfs trên server và client:</p>
<pre>root@nfsv:~# service nfs-kernel-server start
root@nfsv:~# service nfs-kernel-server status
● nfs-server.service - NFS server and services
   Loaded: loaded (/lib/systemd/system/nfs-server.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2018-07-13 17:17:14 +07; 48s ago
 Main PID: 3416 (code=exited, status=0/SUCCESS)

Jul 13 17:17:14 nfsv systemd[1]: Starting NFS server and services...
Jul 13 17:17:14 nfsv exportfs[3413]: exportfs: can't open /etc/exports for reading
Jul 13 17:17:14 nfsv systemd[1]: Started NFS server and services.
Jul 13 17:17:15 nfsv systemd[1]: Started NFS server and services.
Jul 13 17:17:57 nfsv systemd[1]: Started NFS server and services.
root@nfsv:~#</pre>
<p>Trên máy chủ, tôi sẽ tạo ra một thư mục để share các tệp tin và thay đôi quyền sở hữu tệp tin thành không sở hưu bởi ai cả:</p>
<pre>root@nfsv:~# sudo mkdir /var/nfs/general -p
root@nfsv:~# sudo chown nobody:nogroup /var/nfs/general</pre>
<p>Sửa nội dung file <code>/etc/exports</code> để share cả thư mục vừa tạo và thư mục <code>/home</code></p>
<pre>vi /etc/exports
/var/nfs/general 192.168.239.249(rw,sync,no_subtree_check)
/home 192.169.239.249(rw,sync,no_root_squash,no_subtree_check,no_all_squash)</pre>
<p>Trong đó:</p>
<ul>
<li><code>rw</code>: Tùy chọn này cho phép máy tính client truy cập cả đọc và viết vào bộ đĩa (volume).</li>
<li><code>sync</code>: Tùy chọn này bắt buộc NFS phải ghi các thay đổi vào đĩa trước khi trả lời. Điều này dẫn đến một môi trường ổn định và phù hợp hơn kể từ khi trả lời phản ánh tình trạng thực tế của bộ đĩa (volume) từ xa. Tuy nhiên, nó cũng làm giảm tốc độ của hoạt động tập tin.</li>
<li><code>no_subtree_check</code>: tùy chọn này ngăn cản việc kiểm tra cây con, đó là một quá trình mà host phải kiểm tra xem các tập tin thực sự vẫn có sẵn trong cây xuất cho mỗi yêu cầu.</li>
<li><code>no_root_squash</code>: Theo mặc định, NFS chuyển yêu cầu từ người dùng root từ xa vào một người dùng không có đặc quyền trên máy chủ. Điều này đã được dự định như là tính năng bảo mật để ngăn chặn một tài khoản root trên máy khách (client) sử dụng hệ thống tập tin của máy chủ như là root.</li>
<li><code>no_all_squash</code> enables the user’s authority</li>
</ul>
<p>Sau đó khởi động lại server:</p>
<pre><code>systemctl restart nfs-kernel-server
</code></pre>
<p>Điều chỉnh filewall trên server để mở cổng 2049:</p>
<pre><code>sudo ufw allow from 192.168.60.134 to any port nfs
Rules updated
</code></pre>
<p>Trên client:</p>
<pre><code>sudo mkdir -p /nfs/general
sudo mkdir -p /nfs/home 
</code></pre>
<p>mount các thư mục vào client:</p>
<pre><code>mount 192.168.239.250:/var/nfs/general /nfs/general
mount 192.168.239.250:/home /nfs/home	
</code></pre>
<p>Kiểm tra xem đã mount được chưa bằng lệnh df -h. Giờ hãy thử tạo một file mới trong thư mục general trong client hoặc server ta sẽ thấy nó trên máy còn lại.</p>



