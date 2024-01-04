# grafana-prometheus-cadvisor-Tutorial


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

[dockerhub_hub](https://hub.docker.com/repositories/dushyantkumark)

```
docker ps
```

# Node Exporter

It serves as an agent that runs on the machine to be monitored, capturing information about the host's hardware and operating system metrics. These metrics can include details about CPU usage, memory usage, disk utilization, network statistics, and other essential system-level information.

Update docker-compose.yaml in prometheus directory, docker compose file is used to define and run multi-container Docker applications, this time is for "`node-exporter`".

```
version: '3.2'
services:
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    expose:
      - 9100
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

There is only two target "prometheus" and "docker" at all right now , to add new target <> here, add the new job in prometheus.yml for docker containers.

Now open prometheus.yaml file in prometheus directory and add new job_name with static_configs.

```
scrape_configs:
- job_name: node-exporter
  scrape_interval: 5s
  static_configs:
  - targets:
    - node-exporter:9100
```

Now restart prometheus container.

```
docker restart <prometheus containerID/containerNAME>
```


# Grafana Dashboard

got to prometheus -> dashbaord -> add new -> paste the `ID (10619)` for docker.

got to prometheus -> dashbaord -> add new -> paste the `ID (1860)` for node-exporter.
