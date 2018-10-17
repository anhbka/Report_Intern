<h3> Quản lý các images </h3>
<h4>Hiển thị danh sách image </h4>
<p><li> Để liệt kê danh sách các images sử dụng lệnh : glance image-list </li></p>
<pre>root@controller:~# glance image-list
+--------------------------------------+--------+
| ID                                   | Name   |
+--------------------------------------+--------+
| d1019cd9-8dbf-4a0b-a33b-070798c85dab | cirros |
+--------------------------------------+--------+
root@controller:~#</pre>
<p><li>Hiển thị chi tiết thông tin của 1 image sử dụng lệnh: glance image-show image_id</li></p>
<pre>root@controller:~# glance image-show d1019cd9-8dbf-4a0b-a33b-070798c85dab
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290     |
| container_format | bare                                 |
| created_at       | 2018-05-11T07:46:08Z                 |
| disk_format      | qcow2                                |
| id               | d1019cd9-8dbf-4a0b-a33b-070798c85dab |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | cirros                               |
| owner            | 74455f45a933413f81d9c36751c9dfd0     |
| protected        | False                                |
| size             | 13267968                             |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2018-05-11T07:46:09Z                 |
| virtual_size     | None                                 |
| visibility       | public                               |
+------------------+--------------------------------------+</pre>

<h4> Tạo image </h4>
<p>Sử dụng lệnh :</p>
<pre>openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public</pre>
<pre>root@controller:~# openstack image create "cirros1" \
>   --file cirros-0.3.5-x86_64-disk.img \
>   --disk-format qcow2 --container-format bare \
>   --public
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290                     |
| container_format | bare                                                 |
| created_at       | 2018-05-17T09:29:32Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/99c50513-9d32-47cf-b4bf-7eec42339621/file |
| id               | 99c50513-9d32-47cf-b4bf-7eec42339621                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros1                                              |
| owner            | 74455f45a933413f81d9c36751c9dfd0                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13267968                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2018-05-17T09:29:33Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+</pre>
<h4>Upload image</h4>
<p>Trong trường hợp ta tạo ra một image mới và rỗng, ta cần upload dữ liệu cho nó, sử dụng câu lệnh: glance image-upload --file file_name image_id</p>
<pre>root@controller:~# glance image-create --name cirros2 --container bare --disk-format qcow2
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | None                                 |
| container_format | bare                                 |
| created_at       | 2018-05-17T09:32:29Z                 |
| disk_format      | qcow2                                |
| id               | d8be6c30-ad58-48ba-8c54-a406206f6b1f |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | cirros2                              |
| owner            | 74455f45a933413f81d9c36751c9dfd0     |
| protected        | False                                |
| size             | None                                 |
| status           | queued                               |
| tags             | []                                   |
| updated_at       | 2018-05-17T09:32:29Z                 |
| virtual_size     | None                                 |
| visibility       | shared                               |
+------------------+--------------------------------------+</pre>
<h4>Xóa image</h4>
<p> Để xóa image, ta sử dụng câu lệnh : glance image-delete image_id </p>
<pre>root@controller:~# glance image-list
+--------------------------------------+---------+
| ID                                   | Name    |
+--------------------------------------+---------+
| d1019cd9-8dbf-4a0b-a33b-070798c85dab | cirros  |
| 95502eef-fadf-4f58-ad5f-bd3872822ed7 | cirros  |
| 99c50513-9d32-47cf-b4bf-7eec42339621 | cirros1 |
| d8be6c30-ad58-48ba-8c54-a406206f6b1f | cirros2 |
+--------------------------------------+---------+
root@controller:~# glance image-delete d8be6c30-ad58-48ba-8c54-a406206f6b1f
root@controller:~# glance image-list
+--------------------------------------+---------+
| ID                                   | Name    |
+--------------------------------------+---------+
| d1019cd9-8dbf-4a0b-a33b-070798c85dab | cirros  |
| 95502eef-fadf-4f58-ad5f-bd3872822ed7 | cirros  |
| 99c50513-9d32-47cf-b4bf-7eec42339621 | cirros1 |
+--------------------------------------+---------+</pre>
<h4> Thay đổi trạng thái máy ảo </h4>
<p>Như ta đã biết một image upload thành công sẽ ở trạng thái active, người dùng có thể đưa nó về trạng thái deactivate cũng như thay đổi qua lại giữa hai trạng thái bằng câu lệnh :<code> glance image-deactivate <IMAGE_ID>></code> và <code>glance image-reactivate <IMAGE_ID></code><p>
 <pre>root@controller:~# glance image-show d1019cd9-8dbf-4a0b-a33b-070798c85dab
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290     |
| container_format | bare                                 |
| created_at       | 2018-05-11T07:46:08Z                 |
| disk_format      | qcow2                                |
| id               | d1019cd9-8dbf-4a0b-a33b-070798c85dab |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | cirros                               |
| owner            | 74455f45a933413f81d9c36751c9dfd0     |
| protected        | False                                |
| size             | 13267968                             |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2018-05-11T07:46:09Z                 |
| virtual_size     | None                                 |
| visibility       | public                               |
+------------------+--------------------------------------+
root@controller:~#</pre>
 <h3> Sử dụng OpenStack client </h3>
 <p>Giống với glance command line, OpenStack client sử dụng câu lệnh để quản lí glance. Bảng sau đây thể hiện mối quan hệ tương tác giữa hai câu lệnh trên:</p>
  <img src="https://camo.githubusercontent.com/17dbcf7c070792e453760ff7c2a5f99693807740/687474703a2f2f692e696d6775722e636f6d2f65484c753253732e706e67">
  <p> Xem thêm về OpenStack client cho glance tại link sau:
    https://docs.openstack.org/python-openstackclient/latest/</p>
  <p>Cách kiểm tra thông tin trong database</p>
  <p>Bạn có thể connect rồi query ra để xem để hiển thị nội dung database:</p>
  <pre>
root@controller:~# mysql -u root -pWelcome123
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 61
Server version: 10.0.34-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> connect glance
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Connection id:    62
Current database: glance

MariaDB [glance]> select * from image_locations;
+----+--------------------------------------+--------------------------------------------------------------------+---------------------+---------------------+------------+---------+-----------+--------+
| id | image_id                             | value                                                              | created_at          | updated_at          | deleted_at | deleted | meta_data | status |
+----+--------------------------------------+--------------------------------------------------------------------+---------------------+---------------------+------------+---------+-----------+--------+
|  1 | d1019cd9-8dbf-4a0b-a33b-070798c85dab | file:///var/lib/glance/images/d1019cd9-8dbf-4a0b-a33b-070798c85dab | 2018-05-11 07:46:09 | 2018-05-11 07:46:09 | NULL       |       0 | {}        | active |
|  2 | 95502eef-fadf-4f58-ad5f-bd3872822ed7 | file:///var/lib/glance/images/95502eef-fadf-4f58-ad5f-bd3872822ed7 | 2018-05-17 09:29:03 | 2018-05-17 09:29:03 | NULL       |       0 | {}        | active |
|  3 | 99c50513-9d32-47cf-b4bf-7eec42339621 | file:///var/lib/glance/images/99c50513-9d32-47cf-b4bf-7eec42339621 | 2018-05-17 09:29:33 | 2018-05-17 09:29:33 | NULL       |       0 | {}        | active |
+----+--------------------------------------+--------------------------------------------------------------------+---------------------+---------------------+------------+---------+-----------+--------+
3 rows in set (0.00 sec)

MariaDB [glance]></pre>

<h3>Sử dụng cURL</h3>
<h4> Lấy token </h4>
<pre>curl -i -X POST -H "Content-Type: application/json" -d '
{
"auth": {
	"identity": {
		"methods": ["password"],
		"password": {
			"user": {
				"name": "admin",
				"domain": { "name": "Default" },
				"password": "Welcome123"
			}
		}
	},
	"scope": {
		"project": {
			"name": "admin",
			"domain": { "name": "Default" }
		}
	}
}
}' http://localhost:5000/v3/auth/tokens</pre>
<p> Kết quả thu được : </p>
<pre>
HTTP/1.1 201 Created
Date: Thu, 17 May 2018 09:53:13 GMT
Server: Apache/2.4.18 (Ubuntu)
X-Subject-Token: gAAAAABa_VEKc7FsDAuMa8PUKgS4O93lZvX2Jwk8LniNDs07R1ZSY-ihi1gDXwma2GmhIPmROc2DnM6RFO-Wj7FR7cSJDNZSpUhqjt2Th5XknhRPN1sEYn0tDPPBMVs1jVYwNWbqLY28gzNZxAEZYdMCVVW-pMBopsFbnvN03gE9iHuW_qQLHw4
Vary: X-Auth-Token
X-Distribution: Ubuntu
x-openstack-request-id: req-6718081d-1a0b-4e34-89eb-73d92ea81be1
Content-Length: 3289
Content-Type: application/json</pre>
<p>Gán token vừa lấy được vào biến để sử dụng :</p>
<pre>root@controller:~# export OS_AUTH_TOKEN=gAAAAABa_VEKc7FsDAuMa8PUKgS4O93lZvX2Jwk8LniNDs07R1ZSY-ihi1gDXwma2GmhIPmROc2DnM6RFO-Wj7FR7cSJDNZSpUhqjt2Th5XknhRPtDPPBMVs1jVYwNWbqLY28gzNZxAEZYdMCVVW-pMBopsFbnvN03gE9iHuW_qQLHw4</pre>


<h4>List images</h4>
<p>Để liệt kê tất cả các image: Sử dụng phương thức GET truy cập tới endpoint: http://controller:9292/v2/images/</p>
<pre>root@controller:~#  curl -i   http://controller:9292/v2/images \
>  -X GET \
>  -H "X-Auth-Token: $OS_AUTH_TOKEN"
HTTP/1.1 200 OK
Content-Length: 1826
Content-Type: application/json
X-Openstack-Request-Id: req-51205f3a-2381-4977-94d7-d45dbd8f9fbd
Date: Thu, 17 May 2018 10:01:05 GMT

{"images": [{"status": "active", "name": "cirros1", "tags": [], "container_format": "bare", "created_at": "2018-05-17T09:29:32Z", "size": 13267968, "disk_format": "qcow2", "updated_at": "2018-05-17T09:29:33Z", "visibility": "public", "self": "/v2/images/99c50513-9d32-47cf-b4bf-7eec42339621", "min_disk": 0, "protected": false, "id": "99c50513-9d32-47cf-b4bf-7eec42339621", "file": "/v2/images/99c50513-9d32-47cf-b4bf-7eec42339621/file", "checksum": "f8ab98ff5e73ebab884d80c9dc9c7290", "owner": "74455f45a933413f81d9c36751c9dfd0", "virtual_size": null, "min_ram": 0, "schema": "/v2/schemas/image"}, {"status": "active", "name": "cirros", "tags": [], "container_format": "bare", "created_at": "2018-05-17T09:29:01Z", "size": 13267968, "disk_format": "qcow2", "updated_at": "2018-05-17T09:29:03Z", "visibility": "public", "self": "/v2/images/95502eef-fadf-4f58-ad5f-bd3872822ed7", "min_disk": 0, "protected": false, "id": "95502eef-fadf-4f58-ad5f-bd3872822ed7", "file": "/v2/images/95502eef-fadf-4f58-ad5f-bd3872822ed7/file", "checksum": "f8ab98ff5e73ebab884d80c9dc9c7290", "owner": "74455f45a933413f81d9c36751c9dfd0", "virtual_size": null, "min_ram": 0, "schema": "/v2/schemas/image"}, {"status": "active", "name": "cirros", "tags": [], "container_format": "bare", "created_at": "2018-05-11T07:46:08Z", "size": 13267968, "disk_format": "qcow2", "updated_at": "2018-05-11T07:46:09Z", "visibility": "public", "self": "/v2/images/d1019cd9-8dbf-4a0b-a33b-070798c85dab", "min_disk": 0, "protected": false, "id": "d1019cd9-8dbf-4a0b-a33b-070798c85dab", "file": "/v2/images/d1019cd9-8dbf-4a0b-a33b-070798c85dab/file", "checksum": "f8ab98ff5e73ebab884d80c9dc9c7290", "owner": "74455f45a933413f81d9c36751c9dfd0", "virtual_size": null, "min_ram": 0, "schema": "/v2/schemas/image"}], "schema": "/v2/schemas/images", "first": "/v2/images"}root@controller:~# </pre>
<h4> Lọc thông tin image được list ra </h4>
<p>- Các trường thông tin image dùng để lọc thông tin tham khảo <a href="https://docs.openstack.org/developer/glance/glanceapi.html#filtering-images-lists" rel="nofollow">tại đây.</a></p>
<p> - Để lọc thông tin dữ liệu ở đầu ra: ví dụ show ra thông tin các image có tên là cirros, thêm phần query lọc image có trường name là cirros như sau:</p>
<pre>root@controller:~# curl -i   http://controller:9292/v2/images?name=cirros \
>   -X GET \
>   -H "X-Auth-Token: $OS_AUTH_TOKEN"</pre>
<p>- Kết quả </p>
<pre>HTTP/1.1 200 OK
Content-Length: 1251
Content-Type: application/json
X-Openstack-Request-Id: req-d419751b-81f1-4eb9-a0c5-cd24625b1665
Date: Thu, 17 May 2018 10:06:33 GMT

{"images": [{"status": "active", "name": "cirros", "tags": [], "container_format": "bare", "created_at": "2018-05-17T09:29:01Z", "size": 13267968, "disk_format": "qcow2", "updated_at": "2018-05-17T09:29:03Z", "visibility": "public", "self": "/v2/images/95502eef-fadf-4f58-ad5f-bd3872822ed7", "min_disk": 0, "protected": false, "id": "95502eef-fadf-4f58-ad5f-bd3872822ed7", "file": "/v2/images/95502eef-fadf-4f58-ad5f-bd3872822ed7/file", "checksum": "f8ab98ff5e73ebab884d80c9dc9c7290", "owner": "74455f45a933413f81d9c36751c9dfd0", "virtual_size": null, "min_ram": 0, "schema": "/v2/schemas/image"}, {"status": "active", "name": "cirros", "tags": [], "container_format": "bare", "created_at": "2018-05-11T07:46:08Z", "size": 13267968, "disk_format": "qcow2", "updated_at": "2018-05-11T07:46:09Z", "visibility": "public", "self": "/v2/images/d1019cd9-8dbf-4a0b-a33b-070798c85dab", "min_disk": 0, "protected": false, "id": "d1019cd9-8dbf-4a0b-a33b-070798c85dab", "file": "/v2/images/d1019cd9-8dbf-4a0b-a33b-070798c85dab/file", "checksum": "f8ab98ff5e73ebab884d80c9dc9c7290", "owner": "74455f45a933413f81d9c36751c9dfd0", "virtual_size": null, "min_ram": 0, "schema": "/v2/schemas/image"}], "schema": "/v2/schemas/images", "first": "/v2/images?name=cirros"}root@controller:~#</pre>

<h4>Tạo một image mới (chưa upload dữ liệu)</h4>
<li>Để tạo một image mới, ta sử dụng phương thức POST gửi request tới API của Glance như sau:</li>
<pre>root@controller:~# curl -i -X POST -H "X-Auth-Token: $OS_AUTH_TOKEN" \
>     -H "Content-Type: application/json" \
>     -d '{"name": "curl-test", "tags": ["cirros"]}' \
>     http://controller:9292/v2/images</pre>
<p> - Kết quả : </p>
<pre>HTTP/1.1 201 Created
Content-Length: 556
Content-Type: application/json
Location: http://controller:9292/v2/images/049cf087-08f2-403b-8ddc-05be59a7220f
Openstack-Image-Import-Methods: glance-direct,web-download
X-Openstack-Request-Id: req-a7af442a-e6f8-4720-b209-d71b3444bc63
Date: Thu, 17 May 2018 10:10:18 GMT

{"status": "queued", "name": "curl-test", "tags": ["cirros"], "container_format": null, "created_at": "2018-05-17T10:10:18Z", "size": null, "disk_format": null, "updated_at": "2018-05-17T10:10:18Z", "visibility": "shared", "self": "/v2/images/049cf087-08f2-403b-8ddc-05be59a7220f", "min_disk": 0, "protected": false, "id": "049cf087-08f2-403b-8ddc-05be59a7220f", "file": "/v2/images/049cf087-08f2-403b-8ddc-05be59a7220f/file", "checksum": null, "owner": "74455f45a933413f81d9c36751c9dfd0", "virtual_size": null, "min_ram": 0, "schema": "/v2/schemas/image"}root@controller:~#</pre>
<li>Do image này mới tạo chưa được upload dữ liệu nên sẽ trong trạng thái “queued”</li>
<h4>Cập nhật các thuộc tính của image</h4>
<p><li>Để cập nhật các thuộc tính của image, ta sử dụng phương thức PATCH gửi tới API dành riêng cho từng image (Mỗi image được tự động tạo ra một API riêng theo form sau: http://controller:9292/v2/images/<IMAGE_ID> )</li></p>
<p></li>Ví dụ: cập nhật thuộc tính container_format và disk_format của image vừa tạo ta làm như sau:</li></p>
<pre>root@controller:~# curl -i -X POST -H "X-Auth-Token: $OS_AUTH_TOKEN" \
>     -H "Content-Type: application/json" \
>     -d '{"name": "curl-test", "tags": ["cirros"]}' \
>     http://controller:9292/v2/images
HTTP/1.1 201 Created
Content-Length: 556
Content-Type: application/json
Location: http://controller:9292/v2/images/04d2d654-e050-4665-8fc1-3767f08782b8
Openstack-Image-Import-Methods: glance-direct,web-download
X-Openstack-Request-Id: req-e76219b6-d5dd-48e2-9ca0-43c01e09385c
Date: Thu, 17 May 2018 10:17:23 GMT

{"status": "queued", "name": "curl-test", "tags": ["cirros"], "container_format": null, "created_at": "2018-05-17T10:17:23Z", "size": null, "disk_format": null, "updated_at": "2018-05-17T10:17:23Z", "visibility": "shared", "self": "/v2/images/04d2d654-e050-4665-8fc1-3767f08782b8", "min_disk": 0, "protected": false, "id": "04d2d654-e050-4665-8fc1-3767f08782b8", "file": "/v2/images/04d2d654-e050-4665-8fc1-3767f08782b8/file", "checksum": null, "owner": "74455f45a933413f81d9c36751c9dfd0", "virtual_size": null, "min_ram": 0, "schema": "/v2/schemas/image"}root@controller:~#</pre>


<pre>
root@controller:~# curl -i -X PATCH -H "X-Auth-Token: $OS_AUTH_TOKEN" \
> -H "Content-Type: application/openstack-images-v2.1-json-patch" \
> -d '
> [
>     {
>         "op": "add",
>         "path": "/disk_format",
>         "value": "qcow2"
>     },
>     {
>         "op": "add",
>         "path": "/container_format",
>         "value": "bare"
>     }
> ]' http://controller:9292/v2/images/04d2d654-e050-4665-8fc1-3767f08782b8
HTTP/1.1 200 OK
Content-Length: 561
Content-Type: application/json
X-Openstack-Request-Id: req-69dc018c-a812-4b9c-84c2-fc6c9c8b2793
Date: Thu, 17 May 2018 10:18:13 GMT

{"status": "queued", "name": "curl-test", "tags": ["cirros"], "container_format": "bare", "created_at": "2018-05-17T10:17:23Z", "size": null, "disk_format": "qcow2", "updated_at": "2018-05-17T10:18:13Z", "visibility": "shared", "self": "/v2/images/04d2d654-e050-4665-8fc1-3767f08782b8", "min_disk": 0, "protected": false, "id": "04d2d654-e050-4665-8fc1-3767f08782b8", "file": "/v2/images/04d2d654-e050-4665-8fc1-3767f08782b8/file", "checksum": null, "owner": "74455f45a933413f81d9c36751c9dfd0", "virtual_size": null, "min_ram": 0, "schema": "/v2/schemas/image"}root@controller:~# ^C
root@controller:~#</pre>

<h4>Upload dữ liệu lên image</h4>

<pre>curl -i -X PUT -H "X-Auth-Token: $OS_AUTH_TOKEN" \
	-H "Content-Type: application/octet-stream" \
	-d @/root/cirros-0.3.4-x86_64-disk.img \
	http://controller:9292/v2/images/04d2d654-e050-4665-8fc1-3767f08782b8 /file</pre>
	
<h3>Sử dụng REST client</h3>	
<p>Ta có thể sử dụng tiện ích trên Google chrome và Firefox là Advanced REST client:</p>
<p><img src="https://github.com/anhict/images/blob/master/39.jpg"></p>
<p><img src="https://github.com/anhict/images/blob/master/40.jpg"></p>
<p><img src="https://github.com/anhict/images/blob/master/41.jpg"></p>




 
