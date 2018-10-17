<h3>Các ghi chép về cinder</h3>
<li>Có 2 cách sử dụng volume: </li>
<img src="https://github.com/anhict/images/blob/master/cder.png">
<li>Sử dụng để gắn vào máy ảo đã được tạo trước đó: <code>bootable = false</code></li>
<li>Sử dụng để boot máy ảo: <code>bootable = true</code></li>
<li>File chứa trong thư mục <code>/var/lib/cinder/volumes</code> các các file quản lý volume được tạo ra, trong đó có đường dẫn tới volume.</li>
<pre><code>root@cinder:/var/lib/cinder/volumes# cat volume-aa2f2db8-c93b-41ca-9119-d74310caa995

&lt;target iqn.2010-10.org.openstack:volume-aa2f2db8-c93b-41ca-9119-d74310caa995&gt;
    backing-store /dev/cinder-volumes/volume-aa2f2db8-c93b-41ca-9119-d74310caa995
    driver iscsi
    incominguser j5GJW8rYpq5cmd263oXE JCYsTBXrX26ium4F

    write-cache on
&lt;/target&gt;
</code></pre>

<ul>
<li>Kiểm tra các volume trên LVM bằng lệnh:lvs hoặc lsblk </li>
</ul>
<img src="https://github.com/anhict/images/blob/master/cder1.png">
<ul>
<li>
<p>Volume được tạo trên LVM KHÔNG sử dụng cơ chế <code>thin</code> để cấp phát dung lượng lưu trữ (tạo bao nhiêu cấp bấy nhiêu.)</p>
</li>
<li>
<p>Nếu tách máy Cinder thành 1 node (Cinder node) khác và không sử dụng backend thì mặc định volume được tạo ra sẽ lưu tại node cinder.</p>
</li>
<li>
<p>Nếu boot máy ảo từ volume, file máy ảo sẽ nằm trên node Cinder. Node compute sẽ mount tới node cinder thông qua iscsi: </p>
</li>
</ul>
<img src="https://github.com/anhict/images/blob/master/cder2.png">
<h3>Các lệnh về volume</h3>
<ul>
<li>Khởi động các dịch vụ của <code>Cinder</code></li>
</ul>
<div class="highlight highlight-source-shell"><pre><span class="pl-c"><span class="pl-c">#</span> Trên Controller</span>
service cinder-api restart
service cinder-scheduler restart

<span class="pl-c"><span class="pl-c">#</span> Trên Cinder node (nếu triển khai tách node cinder)</span>
service tgt restart
service cinder-volume restart</pre></div>
<ul>
<li>Tạo volume</li>
</ul>
<div class="highlight highlight-source-shell"><pre><span class="pl-c"><span class="pl-c">#</span> Cú pháp đối với OpenStack Mitaka</span>
openstack volume create --size kich_thuoc ten_volume

<span class="pl-c"><span class="pl-c">#</span> Ví dụ tạo volume có kích thước 1Gb và tên là `volume01`</span>
openstack volume create --size 1  volume01 </pre></div>
<ul>
<li>Kiểm tra danh sách các volume</li>
</ul>
<div class="highlight highlight-source-shell"><pre>openstack volume list</pre></div>
<ul>
<li>Gắn volume và gỡ volume khỏi máy ảo</li>
</ul>
<div class="highlight highlight-source-shell"><pre><span class="pl-c"><span class="pl-c">#</span> Cú pháp lệnh gắn volume</span>
openstack server add volume INSTANCE_NAME VOLUME_NAME

<span class="pl-c"><span class="pl-c">#</span> Cú pháp lệnh gỡ volume</span>
openstack server remove volume INSTANCE_NAME VOLUME_NAME

<span class="pl-c"><span class="pl-c">#</span> Trong đó: </span>
 - INSTANCE_NAME: Tên máy ảo
 - VOLUME_NAME: Tên volume

<span class="pl-c"><span class="pl-c">#</span> Ví dụ:</span>
openstack server add volume vm01 volume01 <span class="pl-c"><span class="pl-c">#</span> Gắn vào máy ảo</span>

 openstack server remove volume vm01 vol01-demo <span class="pl-c"><span class="pl-c">#</span> Gỡ ra khỏi máy ảo.</span></pre></div>
















