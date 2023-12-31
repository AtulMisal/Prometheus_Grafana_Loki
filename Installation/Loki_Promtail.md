# Install Grafana on Debian or Ubuntu :

    1. Install Grafana on Debian or Ubuntu
          1. sudo apt-get install -y apt-transport-https
          2. sudo apt-get install -y software-properties-common wget
          3. sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key

    2. Stable release
          1. echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | 
             sudo tee -a /etc/apt/sources.list.d/grafana.list

    3. Beta release (New Updates are installed but not stable)
          1. echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com beta main" | 
             sudo tee -a /etc/apt/sources.list.d/grafana.list

    4. Update the list of available packages
          1. sudo apt-get update

    5. Install the latest OSS release:
          1. sudo apt-get install grafana
          2. sudo /bin/systemctl start grafana-server
          3. sudo /bin/systemctl enable grafana-server  (In case server goes down by this Grafana will automatically starts.)

    6. To start Grafana Server
         1. sudo /bin/systemctl status grafana-server


# Install Loki and Promtail using Docker :
     1. apt-get install docker.io

     2. sudo usermod -aG docker $USER 
        sudo reboot

     3. Download Loki Config :
           1. mkdir grafana_conf
           2. cd gragana_conf
           3. wget https://raw.githubusercontent.com/grafana/loki/v2.8.0/cmd/loki/loki-local-config.yaml -O loki-config.yaml

     4. Run Loki Docker container
           docker run -d --name loki -v $(pwd):/mnt/config -p 3100:3100 grafana/loki:2.8.0 --config.file=/mnt/config/loki-config.yaml

     5. Download Promtail Config
           wget https://raw.githubusercontent.com/grafana/loki/v2.8.0/clients/cmd/promtail/promtail-docker-config.yaml -O 
           promtail-config.yaml

     6. Run Promtail Docker container
           docker run -d --name promtail -v $(pwd):/mnt/config -v /var/log:/var/log --link loki grafana/promtail:2.8.0 
           --config.file=/mnt/config/promtail-config.yaml

# Install Prometheus and cAdvisor :
      cAdvisor (short for container Advisor) analyzes and exposes resource usage and performance data from running containers. 
      cAdvisor exposes Prometheus metrics out of the box.

     1. Download the prometheus config file
           Wget https://raw.githubusercontent.com/prometheus/prometheus/main/documentation/examples/prometheus.yml

     2. Install Prometheus using Docker
           docker run -d --name=prometheus -p 9090:9090 -v <PATH_TO_prometheus.yml_FILE>:/etc/prometheus/prometheus.yml prom/prometheus 
           --config.file=/etc/prometheus/prometheus.yml

     3. Add cAdvisor target
```
scrape_configs:
  - job_name: cadvisor
    scrape_interval: 5s
    static_configs:
      - targets:
         - cadvisor:8080


# Using Docker Compose :

version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
    - cadvisor
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
    - 6379:6379

#  Verify
     1. docker-compose up -d
     2. docker-compose ps

# Test PromQL :
    rate(container_cpu_usage_seconds_total{name="redis"}[1m])
    container_memory_usage_bytes{name="redis"}

