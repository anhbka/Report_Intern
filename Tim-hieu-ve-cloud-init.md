<h3>1. Tổng quan về cloud init </h3>
<p>Cloud-init là một công cụ được sử dụng để thực hiện các thiếp lập ban đầu đối với các máy ảo hóa và cloud. Dịch vụ này sẽ chạy trước quá trình boot, nó lấy dữ liệu từ bên ngoài và thực hiện một số tác động tới máy chủ.</p>
<p>Các tác động mà cloud-init thực hiện phụ thuộc vào loại format thông tin mà nó tìm kiếm được. Các format hỗ trợ:</p>
<ul>
<li>Shell scripts (bắt đầu với #!)</li>
<li>Cloud config files (bắt đầu với #cloud-config)</li>
<li>MIME multipart archive.</li>
<li>Gzip Compressed Content</li>
<li>Cloud Boothook</li>
</ul>
<p>Một trong những định dạng thông dụng nhất dành cho các scripts đó là <code>cloud-config</code>.</p>
<p>cloud-config là các file script được thiết kế để chạy trong các tiến trình cloud-init. Nó được sử dụng cho các cài đặt cấu hình ban đầu trên server như networking, SSH keys, timezone, user data injection...</p>
<p>File config của cloud-init</p>
