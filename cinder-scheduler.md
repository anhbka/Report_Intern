<h3>1. Giới thiệu tổng quan về cinder-scheduler</h3>
<p>Giống với <code>nova-scheduler</code>, cinder cũng có một daemon chịu trách nhiệm cho việc quyết định xem sẽ tạo cinder volume ở đâu khi mô hình có hơn một backend storage. Mặc định nếu người dùng không chỉ rõ host để tạo máy ảo thì cinder-scheduler sẽ thực hiện filter và weight theo những opions sau:</p>
<pre># Which filter class names to use for filtering hosts when not specified in the
# request. (list value)
#scheduler_default_filters = AvailabilityZoneFilter,CapacityFilter,CapabilitiesFilter
# Which weigher class names to use for weighing hosts. (list value)
#scheduler_default_weighers = CapacityWeigher</pre>
<p>Chúng ta sẽ buộc phải kích hoạt tùy chọn <code>filter_scheduler</code> để sử dụng multiple-storage back ends.</p>
<h3>2. Cinder Scheduler Filters</h3>
<ul>
<li>AvailabilityZoneFilter : Filter bằng availability zone</li>
<li>CapabilitiesFilter : Filter theo tài nguyên (máy ảo và volume)</li>
<li>CapacityFilter : Filter dựa vào công suất sử dụng của volume backend</li>
<li>DifferentBackendFilter  : Lên kế hoạch đặt các volume ở các backend khác nhau khi có 1 danh sách các volume</li>
<li>DriverFilter : Dựa vào ‘filter function’ và metrics.</li>
<li>InstanceLocalityFilter : lên kế hoạch cho các volume trên cùng 1 host. Để có thể dùng filter này thì Extended Server Attributes cần được bật bởi nova và user sử dụng phải được khai báo xác thực trên cả nova và cinder.</li>
<li>JsonFilter : Dựa vào JSON-based grammar để chọn lựa backends</li>
<li>RetryFilter : Filter những node chưa từng được schedule</li>
<li>SameBackendFilter : Lên kế hoạch đặt các volume có cùng backend như những volume khác.</li>
</ul>
<h3>3. Cinder Scheduler Weights</h3>
<ul>
<li>AllocatedCapacityWeigher : Allocated Capacity Weigher sẽ tính trọng số của host bằng công suất được phân bổ. Nó sẽ đặt volume vào host được khai báo chiếm ít tài nguyên nhất.</li>
<li>CapacityWeigher : Trạng thái công suất thực tế chưa được sử dụng.</li>
<li>ChanceWeigher : Tính trọng số random, dùng để tạo các volume khi các host gần giống nhau</li>
<li>GoodnessWeigher : Gán trọng số dựa vào goodness function.</li>
</ul>
<p>Goodness rating:</p>
<div class="highlight highlight-source-shell"><pre>0 -- host is a poor choice
<span class="pl-c1">.</span>
<span class="pl-c1">.</span>
50 -- host is a good choice
<span class="pl-c1">.</span>
<span class="pl-c1">.</span>
100 -- host is a perfect choice</pre></div>
<ul>
<li>VolumeNumberWeigher : Tính trọng số theo số lượng volume đang có.</li>
</ul>
<h3>4. Quản lý Block Storage scheduling</h3>
<p>Đối với admin, ta có thể quản lí việc volume sẽ được tạo theo backend nào. Bạn có thể dùng affinity hoặc anti-affinity giữa hai volumes. Affinity có nghĩa rằng chúng được đặt cùng 1 backend và anti-affinity có nghĩa rằng chúng được lưu trên các backend khác nhau.</p>
<p>Một số ví dụ:</p>
<li>Tạo volume cùng backend với Volume_A</li>
<p><code>openstack volume create --hint same_host=Volume_A-UUID --size SIZE VOLUME_NAME</code></p>
<li>Tạo volume khác backend với Volume_A</li>
<code>openstack volume create --hint different_host=Volume_A-UUID --size SIZE VOLUME_NAME</code>
<li>Tạo volume cùng backend với Volume_A và Volume_B</li>
<p><code>openstack volume create --hint same_host=Volume_A-UUID --hint same_host=Volume_B-UUID --size SIZE VOLUME_NAME</code></p>
<p>hoặc</p>
<p><code>openstack volume create --hint same_host="[Volume_A-UUID, Volume_B-UUID]" --size SIZE VOLUME_NAME</code></p>
<li>Tạo volume khác backend với Volume_A và Volume_B</li>
<p><code>openstack volume create --hint different_host=Volume_A-UUID --hint different_host=Volume_B-UUID --size SIZE VOLUME_NAME</code></p>
<p>hoặc</p>
<p><code>openstack volume create --hint different_host="[Volume_A-UUID, Volume_B-UUID]" --size SIZE VOLUME_NAME</code></p>
Tham khảo : https://docs.openstack.org/cinder/latest/cli/cli-cinder-scheduling.html

























