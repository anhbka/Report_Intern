<h3>1. Giới thiệu về nova-scheduler </h3>
<p>Nova Scheduler hỗ trợ filtering và weighting để quyết định instance mới được tạo trên node Compute nào. Scheduler chỉ hỗ trợ nodes Compute.</p>
<p> Tất cả các compute node sẽ public trạng thái của nó bao gồm tài nguyên hiện có và dung lượng phần cứng khả dụng cho nova-scheduler thông qua queue. nova-scheduler sau đó sẽ dựa vào những dữ liệu này để đưa ra quyết định khi có request.</p>
<h3>2. Quá trình filtering và cấu hình </h3>
<img src="https://github.com/anhict/images/blob/master/nova-scheduler1.png">
<p>Cấu hình nova scheduler thông qua file /etc/nova/nova.conf </p>
<pre>scheduler_driver_task_period = 60
scheduler_driver = nova.scheduler.filter_scheduler.FilterScheduler
scheduler_available_filters = nova.scheduler.filters.all_filters
scheduler_default_filters = RetryFilter, AvailabilityZoneFilter, RamFilter, DiskFilter, ComputeFilter, ComputeCapabilitiesFilter, ImagePropertiesFilter, ServerGroupAntiAffinityFilter, ServerGroupAffinityFilter</pre>

<p>Mặc định thì scheduler_driver được cấu hình như là filter scheduler, scheduler này sẽ xem xét các host có đầy đủ các tiêu chí sau:</p>
<ul><li>Chưa từng tham gia vào scheduling (RetryFilter)</li>
<li>Nằm trong vùng requested availability zone (requested availability zone)</li>
<li>Có RAM phù hợp (RamFilter).</li>
<li>Có dung lượng ổ cứng phù hợp cho root và ephemeral storage (DiskFilter).</li>
<li>Có thể thực thi yêu cầu (ComputeFilter)</li>
<li>Đáp ứng các yêu cầu ngoại lệ với các instance type (ComputeCapabilitiesFilter)</li>
<li>Đáp ứng mọi yêu cầu về architecture, hypervisor type, hoặc virtual machine mode properties được khai báo trong instance’s image properties (ImagePropertiesFilter).</li>
<li>Ở host khác với các instance khác trong group (nếu có) (ServerGroupAntiAffinityFilter).</li>
<li>Ở trong danh sách các group hosts (nếu có) (ServerGroupAffinityFilter)</li></ul>
<p>Scheduler sẽ cache lại danh sách available hosts, dùng scheduler_driver_task_period để quy định thời gian danh sách được update.</p>
<p>- Quá trình filter được lặp lại trên các compute nodes. Danh sách các hosts được chọn sẽ được sắp xếp sau bởi weighers. Scheduler sau đó sẽ chọn host theo số lượng các instances request. Nếu scheduler không thể chọn bất cứ host nào thì có nghĩa instance đó không thể được scheduled. Filter Scheduler có rất nhiều cơ chế filtering và weighting. Bạn cũng có thể tự chọn cho mình những giải thuật phù hợp.</p>
<ul><li>AllHostsFilter : Không filter, được tạo máy ảo trên bất cứ host nào available.</li>
<li>ImagePropertiesFilter : filter host dựa vào properties được định nghĩa trên instance’s image. Nó sẽ chọn các host có thể hỗ trợ các thông số cụ thể trên image được sử dụng bởi instance. ImagePropertiesFilter dựa vào kiến trúc, hypervisor type và virtual machine mode được định nghĩa trong instance. Ví dụ, máy ảo yêu cầu host hỗ trợ kiến trúc ARM thì ImagePropertiesFilter sẽ chỉ chọn những host đáp ứng yêu cầu này.</li>
<li>AvailabilityZoneFilter : filter bằng availability zone. Các host phù hợp với availability zone được ghi trên instance properties sẽ được chọn. Nó sẽ xem availability zone của compute node và availability zone từ phần request.</li>
<li>ComputeCapabilitiesFilter : Kiểm tra xem host compute service có đủ khả năng đáp ứng các yêu cầu ngoài lề (extra_specs) với instance type không. Nó sẽ chọn các host có thể tạo được instance type cụ thể. extra_specs chứa key/value pairs ví dụ như free_ram_mb (compared with a number, values like ">= 4096")</li>
<li>ComputeFilter : Chọn tất cả các hosts đang được kích hoạt.</li>
<li>CoreFilter : filter dựa vào mức độ sử dụng CPU core. Nó sẽ chọn host có đủ số lượng CPU core.</li>
<li>AggregateCoreFilter : filter bằng số lượng CPU core với giá trị cpu_allocation_ratio .</li>
<li>IsolatedHostsFilter : filter dựa vào image_isolated, host_isolated và restrict_isolated_hosts_to_isolated_images flags.</li>
<li>JsonFilter : Cho phép sử dụng JSON-based grammar để lựa chọn host.</li>
<li>RamFilter : filter bằng RAM, các hosts có đủ dung lượng RAM sẽ được chọn.</li>
<li>AggregateRamFilter : filter bằng số lượng RAM với giá trị ram_allocation_ratio. ram_allocation_ratio ở đây là tỉ lệ RAM ảo với RAM vật lý (mặc định là 1.5)</li>
<li>DiskFilter : filter bằng dung lượng disk. các hosts có đủ dung lượng disk sẽ được chọn.</li>
<li>AggregateDiskFilter : filter bằng dung lượng disk với giá trị disk_allocation_ratio.</li>
<li>NumInstancesFilter : filter dựa vào số lượng máy ảo đang chạy trên node compute đó. node nào có quá nhiều máy ảo đang chạy sẽ bị loại.</li>
<li>Nếu chỉ số max_instances_per_host đươc thiết lập. Những node có số lượng máy ảo đạt ngưỡng max_instances_per_host sẽ bị ignored.</li>
<li>AggregateNumInstancesFilter : filter dựa theo chỉ số max_instances_per_host.</li>
<li>IoOpsFilter : filter dựa theo số lượng I/O operations.</li>
<li>AggregateIoOpsFilter: filter dựa theo chỉ số max_io_ops_per_host.</li>
<li>SimpleCIDRAffinityFilter : Cho phép các instance trên các node khác nhau có cùng IP block.</li>
<li>DifferentHostFilter : Cho phép các instances đặt trên các node khác nhau.</li>
<li>SameHostFilter : Đặt instance trên cùng 1 node.</li>
<li>RetryFilter : chỉ chọn các host chưa từng được schedule.</li></ul>
<h3>3. Cơ chế weighting và cấu hình </h3>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f553750356d32562e706e67.png">
<p>Sau khi filter các node có thể tạo máy ảo, scheduler sẽ dùng weights để tìm kiếm host phù hợp nhất. Weights được tính toán trên từng host khi mà instance chuẩn bị được schedule, weight được tính toán bằng cách giám sát việc sử dụng tài nguyên của hệ thống. Chúng ta có thể cầu hình để cho các instance được tạo trên các host khác nhau hoặc tạo trên cùng 1 node cho tới khi tài nguyên của node đó cạn kiệt thì mới chuyển sang node tiếp theo.</p>
<p>Nova scheduler tính toán mỗi weight với 1 configurable multiplier rồi sau đó cộng tất cả lại. Host có weight lớn nhất sẽ được ưu tiên. Cơ chế weights cũng cho phép bạn tạo 1 subnet gồm các node phù hợp rồi schedule sẽ lựa chọn ngẫu nhiên.</p>
<img src="https://github.com/anhict/images/blob/master/687474703a2f2f692e696d6775722e636f6d2f624362694c494c2e706e67.png">
<h4>Cấu hình weighting</h4>
<p>- Nếu cells được sử dụng, cells được weighted bởi scheduler tương tự như các hosts.</p>
<p>- Hosts và cells được weighted dựa trên tùy chọn trong file /etc/nova/nova.conf :</p>
<pre>[DEFAULT]
scheduler_host_subset_size = 1
scheduler_weight_classes = nova.scheduler.weights.all_weighers
ram_weight_multiplier = 1.0
io_ops_weight_multiplier = 2.0
soft_affinity_weight_multiplier = 1.0
soft_anti_affinity_weight_multiplier = 1.0
[metrics]
weight_multiplier = 1.0
weight_setting = name1=1.0, name2=-1.0
required = false
weight_of_unavailable = -10000.0</pre>


