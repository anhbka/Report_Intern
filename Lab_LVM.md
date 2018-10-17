<h3>1. Các câu lệnh sử dụng trong LVM</h3>
<p>Đây là những câu lệnh thông dụng để làm việc với LVM</p>
<h4>1.1 Physical volume:</h4>
<p>Câu lệnh với cú pháp pv... đây là những câu lệnh làm việc với physical volume
</p>
<p>Tạo 1 physical volume pvcreate</p>

<p>Hiển thị thuộc tính của physical volume</p>

<pre>pvdisplay</pre>

<p>Điều chỉnh dụng lượng của physical volume</p>
<pre>pvresize</pre>

<p>Xóa physical volume</p>
<pre>pvremove</pre>

<p>Quét tất cả ổ đĩa là physical volume</p>
<pre>pvscan</pre>

<p>Hiển thị các thông số của physical volume</p>
<pre>pvs</pre>
<h4>1.2 Volume group</h4>
<p>Các câu lệnh làm việc với volume group với cú pháp vg.</p>

<p>Tạo volume:</p>
<code>vgcreate [tên các pv để tạo volume group]</code>
<p>Hiển thị các thuộc tính của volume group <code>vgdisplay</code></p>
<p>Mở rộng volume group <code>vgextend [tên pv]</code></p>
<p>Xóa các physical volume ra khỏi volume group <code>vgreduce [tên pv]</code></p>

<p>Xóa volume group <code>vgremove</code> </p>

<p>Xem các thông số của volume group <code>vgs</code></p>
<h4>1.3 Logical volume</h4>
<p>Các câu lệnh làm việc với logical volume với cú pháp lv</p>

<p>Tạo logical volume <code>lvcreate</code></p>
<p>Hiển thị các thuộc tính của logical volume <code>lvdisplay</code></p>

<p>Mở rộng logical volume. Lưu ý: chỉ mở rộng khi volume group còn dung lượng trống <code>lvextend</code></p>

<p>Xóa logical volume khỏi volume group <code>lvremove</code></p>

<p>Thay đổi dung lượng logical volume <code>lvresize</code></p>
<h3.2. Lab LVM</h3>
<p>Sử dụng VMware, HĐH Ubuntu Server 14.04</p>
<p>Tiến hành thêm các ổ cứng vào như hình :</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_29.png">
<p>Bạn có thể kiểm tra xem có những Hard Drives nào trên hệ thống bằng cách sử dụng câu lệnh lsblk.</p>
<pre>lsblk</pre>
<img src="https://github.com/anhict/images/blob/master/Screenshot_30.png">
<p>Từ các Hard Drives trên hệ thống, bạn tạo các partition. Ở đây, từ sdb, mình tạo các partition bằng cách sử dụng lệnh sau fdisk /dev/sdb</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_31.png">
<img src="https://github.com/anhict/images/blob/master/Screenshot_34.png">
<p>Trong đó :</p>
<p>Chọn "n" để bắt đầu tạo partition</p>
<p>Chọn "p" để tạo partition primary</p>
<p>Chọn "1" để tạo partition primary 1</p>
<p>Tại "First sector (2048-20971519, default 2048)"" để mặc định</p>
<p>Tại "Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519)"" bạn chọn "+10G" để partition bạn tạo ra có dung lượng 10G
<p>chọn "w" để lưu lại và thoát</p>
<p>Tiếp theo bạn thay đổi định dạng của partition vừa mới tạo thành LVM</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_35.png">
<p>Trong đó:</p>
<p>Bạn chọn "t" để thay đổi định dạng partition</p>
<p>Bạn chọn "8e" để đổi thành LVM</p>
<p>chọn "w" để lưu lại và thoát.</p>
<p>B3. Tạo Physical Volume</p>
<p>Tạo các Physical Volume là /dev/sdb1 ,/dev/sdb2và /dev/sdc1, /dev/sdc2 bằng các lệnh sau:</p>
<img src="https://github.com/anhict/images/blob/master/Screenshot_36.png">
<p>Có thể kiểm tra các Physical Volume bằng câu lệnh pvs hoặc có thể sử dụng lệnh pvdisplay.</p>

<p>B4. Tạo Volume Group</p>
<p>Tiếp theo, nhóm các Physical Volume thành 1 Volume Group bằng cách sử dụng câu lệnh sau:</p>
<pre>vgcreate vg-demo1 /dev/sdb1 /dev/sdb2 /dev/sdc1 /dev/sdc2</pre>
<img src="https://github.com/anhict/images/blob/master/Screenshot_37.png">
<p>Trong đó vg-demo1 là tên của Volume Group</p>

<p>Có thể sử dụng câu lệnh sau để kiểm tra lại các Volume Group đã tạo</p>
<pre># vgs
# vgdisplay</pre>
<img src="https://github.com/anhict/images/blob/master/Screenshot_38.png">
<p>B5. Tạo Logical Volume</p>
<p>Từ một Volume Group, chúng ta có thể tạo ra các Logical Volume bằng cách sử dụng lệnh sau:</p>

<pre># lvcreate -L 1G -n lv-demo1 vg-demo1</pre>
<img src="https://github.com/anhict/images/blob/master/Screenshot_39.png">
<p>Trong đó :</p>

<p>L: Chỉ ra dung lượng của logical volume</p>
<p>-n Chỉ ra tên logical volume</p>
<p>lv-demo1 là tên Logical Volume</p>
<p>vg-demo1 là Volume Group mà mình vừa tạo ở bước trước</p>
<p>Lưu ý là chúng ta có thể tạo nhiều Logical Volume từ 1 Volume Group</p>
<p>Có thể sử dụng câu lệnh sau để kiểm tra lại các Logical Volume đã tạo :</p>
<pre.# lvs
# lvdisplay</pre>
<img src="https://github.com/anhict/images/blob/master/Screenshot_40.png">
<p>B6. Định dạng Logical Volume :</p>
<p>Để format các Logical Volume thành các định dạng như ext2, ext3, ext4, ta có thể làm như sau:</p>
<pre>mkfs -t ext4 /dev/vg-demo1/lv-demo1</pre>
<img src="https://github.com/anhict/images/blob/master/Screenshot_41.png">
