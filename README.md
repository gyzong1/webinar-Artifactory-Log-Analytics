# 描述
* 配置 Artifactory + Fluentd + Prometheus + Grafana 实现日志分析
# 内容
* 安装各组件  
* 日志分析展示
# 步骤
## 安装 Artifactory  
* 访问 Artifactory 下载地址 https://jfrog.com/download-legacy/?product=artifactory&installer=linux， 下载Artifactory  
<img width="1751" alt="image" src="https://github.com/gyzong1/webinar-Artifactory-Log-Analytics/blob/main/images/Artifactory_Download.png">  

* 解压并启动  
```bash  
tar zxf jfrog-artifactory-pro-7.41.12-linux.tar.gz  
cd artifactory-pro-7.41.12/app/bin/  
./artifactoryctl start  
``` 


## 安装 td-agent  
```bash
curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent4.sh | sh
# 配置路径：/etc/td-agent/
# 日志路径：/var/log/td-agent/
```

## 安装 prometheus 插件  
```bash
sudo td-agent-gem install fluent-plugin-prometheus
```

## 修改 td-agent.conf 以搜集日志
* 下载 github 上的 fluent.conf.rt  
```bash
git clone https://github.com/jfrog/log-analytics-prometheus.git  
cp log-analytics-prometheus/fluent.conf.rt /etc/td-agent/td-agent.conf

# 修改启动td-agent的用户、组，添加环境变量
vim /usr/lib/systemd/system/td-agent.service
[Service]
User=root
Group=root
Environment=JF_PRODUCT_DATA_INTERNAL=/root/webinar/artifactory-pro-7.41.12/var
```  
* 访问：http://ip:24231/metrics, 查看日志metrics信息
<img width="1751" alt="image" src="https://github.com/gyzong1/webinar-Artifactory-Log-Analytics/blob/main/images/Artifactory_Log_Metrics.png"> 


## 安装 Prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.39.0/prometheus-2.39.0.linux-amd64.tar.gz
tar zxf prometheus-2.39.0.linux-amd64.tar.gz
```
添加服务启动
```bash
vim /usr/lib/systemd/system/prometheus-server.service
[Unit]
Description=prometheus-server
After=network.target
 
[Service]
Type=simple
User=root
ExecStart=/root/webinar/prometheus-2.39.0.linux-amd64/prometheus --config.file=/root/webinar/prometheus-2.39.0.linux-amd64/prometheus.yml
Restart=on-failure
 
[Install]
WantedBy=multi-user.target

systemctl start prometheus-server
```
* 访问：http://ip:9090, 确认 Fluentd 连接正常
<img width="1751" alt="image" src="https://github.com/gyzong1/webinar-Artifactory-Log-Analytics/blob/main/images/Prometheus_Status.png"> 

## 安装 Grafana
```bash
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-9.2.0~beta1-1.x86_64.rpm
yum install -y grafana-9.1.7-1.x86_64.rpm
systemctl start grafana
```
* 访问：http://ip:3000
默认用户名/密码：admin/admin

## 选择数据源
* "Settings"-->"Data sources"-->"Add data source"-->"Prometheus"，填写URL信息
<img width="1751" alt="image" src="https://github.com/gyzong1/webinar-Artifactory-Log-Analytics/blob/main/images/Grafana_Data_Sources1.png"> 
<img width="1751" alt="image" src="https://github.com/gyzong1/webinar-Artifactory-Log-Analytics/blob/main/images/Grafana_Data_Sources2.png"> 

## 导入 Dashboard
* dashboard文件路径：log-analytics-prometheus/grafana/Artifactory-dashboard.json
<img width="1751" alt="image" src="https://github.com/gyzong1/Uwebinar-Artifactory-Log-Analytics/blob/main/images/Artifactory_Download.png"> 

## 查看数据
<img width="1751" alt="image" src="https://github.com/gyzong1/webinar-Artifactory-Log-Analytics/blob/main/images/Maven-1.png"> 
