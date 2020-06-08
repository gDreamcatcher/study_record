# 服务器监控

https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
tar -xf node_exporter-1.1.2.linux-amd64.tar.gz
cd node_exporter-1.1.2.linux-amd64 && ./node_exporter

https://github.com/prometheus/prometheus/releases/download/v2.26.0/prometheus-2.26.0.linux-amd64.tar.gz
tar -xf prometheus-2.26.0.linux-amd64.tar.gz
cd prometheus-2.26.0.linux-amd64 
cat > prometheus.yml << EOF
global:
  scrape_interval: 15s

scrape_configs:
- job_name: node
  static_configs:
  - targets: ['localhost:9100']

EOF
./prometheus --config.file=./prometheus.yml


wget https://dl.grafana.com/oss/release/grafana-7.5.4.linux-amd64.tar.gz
tar -zxvf grafana-7.5.4.linux-amd64.tar.gz
cd grafana-7.5.4 && ./bin/grafana-server

dashboard id: 1860



进程监控
wget https://github.com/ncabatoff/process-exporter/releases/download/v0.7.5/process-exporter-0.7.5.linux-386.tar.gz
tar -xf process-exporter-0.7.5.linux-386.tar.gz && cd process-exporter-0.7.5.linux-386
cat > process-names.yaml << EOF
process_names:
  - name: wardrobe
    exe:
    - /usr/bin/java
  - name: shortvideo
    exe:
    - /export/servers/java/bin/java

EOF
./process-export --config.path process-names.yaml
