# grafana-prometheus-cadvisor-Tutorial

https://krishkp.hashnode.dev/monitoring-in-prometheus-and-grafana-docker-dashboard-cadvisor

this is a documentation of how to setup prometheus , grafana via cadvisor in linux.

# Install Grafana on Debian or Ubuntu

sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key

Stable release
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Update the list of available packages

sudo apt-get update

# Install the latest OSS release:

sudo apt-get install grafana

# To start Grafana Server

sudo /bin/systemctl status grafana-server

sudo systemctl start grafana-server

sudo systemctl enable grafana-server

# Install Prometheus and cAdvisor

cAdvisor (short for container Advisor) analyzes and exposes resource usage and performance data from running containers. cAdvisor exposes Prometheus metrics out of the box.

# Download the prometheus config file

wget https://raw.githubusercontent.com/prometheus/prometheus/main/documentation/examples/prometheus.yml

**after this, a prometheus.yml file will be added in the directory. this is prometheus config file where we have to add the other scrap configurations.**
like # Add cAdvisor target

```
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
```

# Install Prometheus using Docker

docker run -d --name=prometheus -p 9090:9090 -v <PATH_TO_prometheus.yml_FILE>:/etc/prometheus/prometheus.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml

# Using Docker Compose


```
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
```


# Verifydocker

```
docker-compose up -d
docker-compose ps
```

# Test PromQL

```
rate(container_cpu_usage_seconds_total{name="redis"}[1m])
container_memory_usage_bytes{name="redis"}
```

# Install Tetris Game using Docker

docker run -d -p 80:80 --name tetris-game dushyantkumark/tetris-v2:latest

[dockerhub_url](https://hub.docker.com/repositories/dushyantkumark)

```
docker ps
```
