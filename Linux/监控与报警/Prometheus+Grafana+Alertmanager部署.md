#  Prometheus+Grafana+Alertmanager部署

## 软件介绍

### 1. **Prometheus**

Prometheus 是一个开源的监控系统，主要用于捕捉和存储时间序列数据。它能够通过拉取（pull）模型从目标上获取数据，支持灵活的查询语言 **PromQL**，并提供强大的可视化和报警功能。

### 2. **Pushgateway**

**Pushgateway** 是 Prometheus 生态系统中的一个工具，专门用来处理短期任务或批处理作业的指标。这些短期任务通常是无法直接通过 Prometheus 的 pull 模型抓取的，因此 Pushgateway 可以接收来自这些任务的指标数据并将其保存在 Prometheus 可抓取的形式中。
主要功能：

- 允许临时的、短期任务将其指标推送到 Pushgateway。
- 支持长期运行的指标的暴露，适合批处理任务的场景。

### 3. **Node Exporter**

**Node Exporter** 是 Prometheus 生态系统中用于收集 Linux/Unix 系统层面指标的组件。它暴露系统硬件和操作系统的运行状况，例如：

- CPU 使用率
- 内存使用情况
- 磁盘 I/O
- 网络流量

### 4. **Grafana**

**Grafana** 是一个开源的数据可视化工具，能够与 Prometheus 等数据源集成。它通过直观的图表、仪表盘（dashboard）展示时间序列数据，方便用户监控系统性能和运行状态。
主要功能：

- 支持多种数据源（包括 Prometheus、Elasticsearch、MySQL 等）
- 自定义仪表盘和报警
- 提供丰富的插件以扩展功能

### 5. **Alertmanager**

**Alertmanager** 是 Prometheus 的报警管理组件，用于处理来自 Prometheus 服务器的报警。它能够根据预定义的规则分组、路由、抑制和消除报警。Alertmanager 可以将报警发送到各种通知渠道，如邮件、Slack、PagerDuty 等。
主要功能：

- 灵活的报警管理和路由规则
- 报警抑制与分组
- 支持多种通知方式



这些组件结合在一起，形成一个强大的监控系统，Prometheus 负责收集和存储数据，Pushgateway 用于处理无法直接拉取的任务，Node Exporter 提供系统层面的指标，Grafana 实现数据的可视化，Alertmanager 管理报警。



## 实例环境与下载

### 实例环境

本文搭建的 LNMP 环境软件组成版本及说明如下：

- Linux：Linux 操作系统，本文以 debian 12.6 为例。
- prometheus：本文以  prometheus 2.54.1 为例。
- pushgateway：本文以 pushgateway 1.10.0 为例。
- node-exporter：本文以 node-exporter 1.8.2 为例。
- Grafana：本文以 Grafana 11.2.0 为例。
- Alertmanager：本文以 Alertmanager 0.27.0 为例。

### 下载地址

部分软件有国内加速下载请根据需求选择下载方式，下载地址如下：

> prometheus：
>
> ​	github：https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz
>
> ​	清华源镜像：https://mirrors.tuna.tsinghua.edu.cn/github-release/prometheus/prometheus/2.54.1%20_%202024-08-27/prometheus-2.54.1.linux-amd64.tar.gz
>
> 
>
> pushgateway（github）：https://github.com/prometheus/pushgateway/releases/download/v1.10.0/pushgateway-1.10.0.linux-amd64.tar.gz
>
> node-exporter（github）:https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
>
> Alertmanager（github）：https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
>
> Grafana：
>
> ​	华为源：https://repo.huaweicloud.com/grafana/11.2.0/grafana-enterprise_11.2.0_amd64.deb
>
> ​	grafana官网：https://dl.grafana.com/enterprise/release/grafana-enterprise_11.2.2+security~01_amd64.deb

> 注意：由于各软件会升级迭代，请根据实际情况调整下载地址



进入服务器，执行如下命令下载并解压：

```shell

cd /opt
mkdir prometheus_env
cd prometheus_env
wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz
wget https://github.com/prometheus/pushgateway/releases/download/v1.10.0/pushgateway-1.10.0.linux-amd64.tar.gz
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_11.2.2+security~01_amd64.deb
tar -zxvf prometheus-2.54.1.linux-amd64.tar.gz
tar -zxvf pushgateway-1.10.0.linux-amd64.tar.gz
tar -zxvf node_exporter-1.8.2.linux-amd64.tar.gz
tar -zxvf alertmanager-0.27.0.linux-amd64.tar.gz

sudo apt-get install -y adduser libfontconfig1 musl
sudo dpkg -i grafana-enterprise_11.2.2_amd64.deb


//修改文件名去掉版本号
cd /opt/prometheus_env
for i in `ls -l|awk '{print $9}'`;do new_name=$(echo "$i" | awk -F '-' '{print $1}');mv "$i" "$new_name";done
```



## 服务配置

### 1.修改prometheus.yml 配置文件

```shell
cd /opt/prometheus_env/prometheus
vi prometheus.yml
```

内容如下：

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           - 127.0.0.1:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
   - "alarm_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
        labels:
          instance: 'prometheus'
  - job_name: 'linux'
    static_configs:
      - targets: ['127.0.0.1:9100']
        labels:
          instance: 'localhost'
  - job_name: 'pushgateway'
    static_configs:
      - targets: ['127.0.0.1:9091']
        labels:
          instance: 'pushgateway'
```



#### 配置解析：

##### **全局配置（global）**：

- `scrape_interval` 和 `evaluation_interval` 设置为 15 秒，意味着每 15 秒抓取一次指标数据并评估报警规则。

##### **报警管理（alerting）**：

- 配置 Alertmanager 的地址（`127.0.0.1:9093`），确保 Alertmanager 正在该地址上运行。

##### **规则文件（rule_files）**：

- 指向 `alarm_rules.yml`，请确保该文件存在且格式正确。

##### **抓取配置（scrape_configs）**：

- **Prometheus 本身**：抓取自身的指标，目标为 `127.0.0.1:9090`。
- **Linux 监控**：通过 Node Exporter 抓取本地系统指标，目标为 `127.0.0.1:9100`。
- **Pushgateway**：监控 Pushgateway 的指标，目标为 `127.0.0.1:9091`。

根据需要调整 `scrape_interval` 和 `evaluation_interval`，如果你希望更频繁地抓取数据，可以将其缩短。如果有多个实例，可以考虑使用服务发现机制。



### 2.新增alarm_rules.yml 文件

```shell
vim alarm_rules.yml

groups:
- name: node
  rules:
  - alert: server_status
    expr: up{} == 0 
    for: 15s
    annotations:
      summary: "机器{{ $labels.instance }} 挂了"
      description: "请立即查看问题!"
  - alert: server_status
    expr: 100 - ((node_memory_MemAvailable_bytes * 100) / node_memory_MemTotal_bytes) > 40     
    for: 1s
    annotations:
      summary: "机器{{ $labels.instance }} 内存大于50%"
      description: "请立即查看问题!"
  - alert: server_status
    expr: (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[2m])) by (instance)) * 100 > 70  
    for: 1s
    annotations:
      summary: "机器{{ $labels.instance }} CPU使用率大于70%"
      description: "请立即查看问题!"        
  - alert: server_status
    expr: max((node_filesystem_size_bytes{fstype=~"ext.?|xfs"}-node_filesystem_free_bytes{fstype=~"ext.?|xfs"}) *100/(node_filesystem_avail_bytes {fstype=~"ext.?|xfs"}+(node_filesystem_size_bytes{fstype=~"ext.?|xfs"}-node_filesystem_free_bytes{fstype=~"ext.?|xfs"})))by(instance) > 80  
    for: 15s
    annotations:
      summary: "机器{{ $labels.instance }} 分区使用率大于80%"
      description: "请立即查看问题!"
```



### 3.修改alertmanager.yml 文件

```shell
cd /opt/prometheus_env/alertmanager
vim alertmanager.yml 
```

内容如下：

```
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.163.com:465' # 定义163邮箱服务器端
  smtp_from: 'yourmail@163.com'  #来自哪个邮箱发的
  smtp_auth_username: 'yourmail@163.com' #邮箱验证
  smtp_auth_password: '1111aaaa'   # 邮箱授权码，不是登录密码
  smtp_require_tls: true   # 是否启用tls
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 3m
  receiver: 'email'
receivers:
  - name: 'email'
    email_configs:
      - to: 'a975707253@qq.com'
#inhibit_rules:
#  - source_match:
#      severity: 'critical'
#    target_match:
#      severity: 'warning'
#    equal: ['alertname', 'dev', 'instance']
```

#### 配置解析：

##### global

- **resolve_timeout**: 设置告警的解决时间，5分钟内未解决则会被视为已解决。
- **smtp_smarthost**: 定义 SMTP 服务器的地址和端口。
- **smtp_from**: 指定发送邮件时显示的发件人邮箱地址。
- **smtp_auth_username**: SMTP 认证的用户名，一般是完整的邮箱地址。
- **smtp_auth_password**: 邮箱的授权码，而不是登录密码，需要在邮箱设置中生成。
- **smtp_require_tls**: 是否要求使用 TLS 进行加密传输。

##### route

- **group_by**: 定义在同一组内如何聚合告警，这里按 `alertname` 聚合。
- **group_wait**: 在发送组告警前的等待时间，10秒。
- **group_interval**: 组内告警发送的最小间隔，10秒。
- **repeat_interval**: 如果告警没有解决，重复发送告警的最小时间间隔，这里设置为 3 分钟。
- **receiver**: 指定告警的接收者，这里是 `mail`。

##### receivers

- **name**: 定义接收者的名称，这里为 `mail`，与 `route` 中的接收者一致。
- email_configs:邮件发送的配置。
  - **to**: 接收邮件的地址，可以列出多个邮箱地址，使用换行分隔。



### 4.修改defaults.ini文件

修改grafana的defaults.ini文件将界面可以匿名访问

```shell
vim /etc/grafana/grafana.ini
```



```shell
#################################### Anonymous Auth ######################
[auth.anonymous]
# enable anonymous access
enabled = true
```



### 5.新增systemd的service

```shell
cd /usr/lib/systemd/system
```

##### pushgateway.service文件

```shell
[Unit]
Description=Prometheus Push Gateway
After=network.target

[Service]
ExecStart=/opt/prometheus_env/pushgateway/pushgateway

User=root
[Install]
WantedBy=multi-user.target
```



##### node_exporter.service文件

```shell
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
ExecStart=/opt/prometheus_env/node_exporter/node_exporter

User=root
[Install]
WantedBy=multi-user.target
```



##### prometheus.service文件

```shell
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
ExecStart=/opt/prometheus_env/node_exporter/node_exporter

User=root
[Install]
WantedBy=multi-user.target
ops@VM-16-17-debian:/usr/lib/systemd/system$ cat prometheus.service 
[Unit]
Description=Prometheus Service
After=network.target

[Service]
ExecStart=/opt/prometheus_env/prometheus/prometheus \
--config.file=/opt/prometheus_env/prometheus/prometheus.yml \
--web.read-timeout=5m  \
--web.max-connections=10 \
--storage.tsdb.retention=15d \
--storage.tsdb.path=/prometheus/data \
--query.max-concurrency=20 \
--query.timeout=2m

User=root
[Install]
WantedBy=multi-user.target
```

##### alertmanager.service文件

```shell
[Unit]
Description=Prometheus alertmanager
After=network.target

[Service]
ExecStart=/opt/prometheus_env/alertmanager/alertmanager \
--storage.path=/opt/prometheus_env/alertmanager/data \
--config.file=/opt/prometheus_env/alertmanager/alertmanager.yml

User=root
[Install]
WantedBy=multi-user.target
```



## 启动

```shell
重载配置：
systemctl daemon-reload

开启服务：
systemctl start pushgateway
systemctl start node_exporter
systemctl start prometheus
systemctl start grafana
systemctl start alertmanager

设置开机启动：
systemctl enable pushgateway
systemctl enable node_exporter
systemctl enable prometheus
systemctl enable grafana
systemctl enable alertmanager

查看服务状态:
systemctl status pushgateway
```

