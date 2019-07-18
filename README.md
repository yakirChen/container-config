# docker compose

## 同步系统时间
yum install ntpdate
ntpdate -u edu.ntp.org.cn


## Canal

+ canal maven build
```bash
mvn clean package -DskipTests -U -Denv=release
```


## Elasticsearch

### Important

+ [Important System Configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html)

+ [Important Elasticsearch configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)

+ Disable swapping
  * `sudo swapoff -a`
  * `conf/elasticsearch.yml` 添加 `bootstrap.memory_lock: true`
    - 检查[http://192.168.1.149:9200/_nodes?filter_path=\*\*.mlockall](http://192.168.1.149:9200/_nodes?filter_path=**.mlockall) 返回类似json, `mlockall`为`true`
    ```json
    {"nodes":{"BOjAhFPyT-SSxV5p6PSOpg":{"process":{"mlockall":true}},"9dFKHlwoQFSNzaInhuUM2g":{"process":{"mlockall":true}}}}
    ```

+ File Descriptors & Number of threads
[http://192.168.1.149:9200/_nodes/stats/process?filter_path=\*\*.max_file_descriptors](http://192.168.1.149:9200/_nodes/stats/process?filter_path=**.max_file_descriptors)
```json
{"nodes":{"9dFKHlwoQFSNzaInhuUM2g":{"process":{"max_file_descriptors":1048576}},"BOjAhFPyT-SSxV5p6PSOpg":{"process":{"max_file_descriptors":1048576}}}}
```
  * macOS `-XX:-MaxFDLimit`
  * Linux
    `sudo vim /etc/security/limits.conf`
    ```bash
    * soft nofile 262144
    * hard nofile 262144
    * soft nproc 65535
    * hard nproc 65535
    * soft memlock unlimited
    * hard memlock unlimited
    ```
    `sudo systemctl daemon-reload`

+ Virtual memory
  `sudo vim /etc/sysctl.conf`
  ```bash
  vm.max_map_count=1048576
  ```
  _**`sudo reboot`**_(be careful)  
  检查`sysctl -p`

+ DNS cache settings
```properties
-Des.networkaddress.cache.ttl=60
-Des.networkaddress.cache.negative.ttl=10
```

+ Java Native Access (JNA)
```properties
# 默认挂载 /tmp
-Djna.tmpdir=<path>
-Djna.nosys=true # 待研究这个参数
```
+ Single-node discovery & Discovery configuration check
```
discovery.type=single-node

# 强制 bootstrap checks
# conf/elasticsearch.yml
es.enforce.bootstrap.checks: true
# jvm options
-Des.enforce.bootstrap.checks=true
```

```yaml
# conf/elasticsearch.yml
discovery.seed_hosts
discovery.seed_providers
cluster.initial_master_nodes
```

+ 分片分配(Shard Allocation)
```yaml
# all, primaries, new_primaries, none
cluster.routing.allocation.enable: all


# all, primaries, replicas, none
cluster.routing.rebalance.enable: all
# always, indices_primaries_active, indices_all_active
cluster.routing.allocation.allow_rebalance: always

```

### Node

+ Master Eligible Nodeedit
没设置`x-pack`
```yaml
node.master: true 
node.data: false 
node.ingest: false 
cluster.remote.connect: false
```
设置了`x-pack`
