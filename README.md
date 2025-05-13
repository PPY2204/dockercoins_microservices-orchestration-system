
# 🐳 Docker Swarm Microservices System – Monitoring & Orchestration

![Docker Swarm](https://img.shields.io/badge/Docker-Swarm-2496ED?logo=docker)
![Status](https://img.shields.io/badge/Status-80%25%20Complete-yellowgreen)

This project demonstrates a **production-ready microservices system** deployed using **Docker Swarm** for orchestration, with centralized logging, monitoring, high availability (HA), auto-scaling, and load balancing. It simulates a scalable distributed environment optimized for performance and resilience.

## 📌 Project Overview

Key features:

- **Container Orchestration**: Docker Swarm for service deployment and management
- **Centralized Monitoring**: Prometheus, Grafana, cAdvisor, node_exporter, and InfluxDB
- **Distributed Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **High Availability**: Multi-manager setup for fault tolerance
- **Auto-Scaling**: Manual scaling with plans for metric-based automation
- **Load Balancing**: Swarm's built-in routing for traffic distribution

## 🚀 Technology Stack

### Core Components

| Component     | Purpose                 | Version |
| ------------- | ----------------------- | ------- |
| Docker Swarm  | Container orchestration | 24.10   |
| Ubuntu Server | Host OS                 | 24.10   |

### Monitoring Stack

| Tool           | Function                   | Port |
| -------------- | -------------------------- | ---- |
| Prometheus     | Metrics collection         | 9090 |
| Grafana        | Visualization              | 3000 |
| cAdvisor       | Container metrics          | 8080 |
| node_exporter  | Host metrics               | 9100 |
| InfluxDB       | Time-series DB for metrics | 8086 |

### Logging Stack

| Component     | Function          | Port |
| ------------- | ----------------- | ---- |
| Elasticsearch | Log storage       | 9200 |
| Logstash      | Log processing    | 5044 |
| Kibana        | Log visualization | 5601 |

## 🛠️ System Setup (Virtual Environment)

### 1. Virtual Machine Setup

Created 10 Ubuntu 24.10 VMs in VirtualBox:

- **3 Manager Nodes (HA)**:
  - master01 (192.168.100.10)
  - master02 (192.168.100.11)
  - master03 (192.168.100.12)
- **7 Worker Nodes**:
  - worker01–worker07 (192.168.100.13–19)

### 2. Software Installation

```bash
sudo apt update && sudo apt install -y \
    ifupdown \
    net-tools \
    openssh-server \
    docker.io \
    docker-compose
```

### 3. Network Configuration

Each VM has two network adapters:

- **NAT Adapter**: Internet access (Gateway: 10.0.2.2)
- **Host-only Adapter**: Internal communication (Subnet: 192.168.100.0/24)

**Static IP Example (master01):**

```bash
auto lo
iface lo inet loopback

auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
    address 192.168.100.10
    netmask 255.255.255.0
```

**Validation:**

```bash
ping www.google.com           # Internet access
ping 192.168.100.11           # Internal communication
ssh osboxes@192.168.100.10    # SSH from host
```

## ⚙️ System Architecture

### 📌 Services

| Service | Description                       | Port |
| ------- | --------------------------------- | ---- |
| rng     | Generates random bytes (HTTP)     | 8001 |
| hasher  | Computes hash of POSTed data      | 8002 |
| worker  | Background service (calls rng...) | -    |
| redis   | Stores hash results               | 6379 |
| webui   | Visualizes blockchain-like data   | 8000 |

### 🔗 Communication Flow

- `worker` calls `rng` (HTTP GET) and `hasher` (HTTP POST)
- `worker` and `webui` share state via `redis`
- `webui` serves users on port 8000
- All services communicate over an **encrypted overlay network**

### 📦 Docker Images

Located in `~/orchestration-workshop/dockercoins`:

- `redis`: Official redis:alpine image
- `rng`, `hasher`, `webui`, `worker`: Custom-built images

### 🚀 Docker Swarm Deployment

1. **Initialize Swarm (on master01):**

```bash
docker swarm init --advertise-addr 192.168.100.10
```

2. **Join Workers (using token):**

```bash
docker swarm join --token <token> 192.168.100.10:2377
```

3. **Promote Managers:**

```bash
docker node promote master02 master03
```

> ✅ HA Note: 3 managers ensure fault tolerance (quorum: 2n+1). If one manager fails, the swarm remains operational.

4. **Create Overlay Network:**

```bash
docker network create \
  --driver overlay \
  --attachable \
  --opt encrypted \
  coinswarmnet
```

5. **Build and Push Images (example - rng):**

```bash
cd ~/orchestration-workshop/dockercoins/rng
docker build -t ismartio/rng:latest .
docker push ismartio/rng:latest
```

Repeat for other services.

6. **Deploy Services:**

```bash
docker service create \
  --name rng \
  --network coinswarmnet \
  --replicas 3 \
  --publish published=8001,target=80 \
  ismartio/rng:latest
```

### docker-compose.yml (Full Stack):

```yaml
version: '3.8'
services:
  redis:
    image: redis:alpine
    networks:
      - coinswarmnet
    deploy:
      replicas: 3
      placement:
        constraints: [node.role == worker]
    ports:
      - "6379:6379"

  rng:
    image: ismartio/rng:latest
    networks:
      - coinswarmnet
    deploy:
      replicas: 3
    ports:
      - "8001:80"

  hasher:
    image: ismartio/hasher:latest
    networks:
      - coinswarmnet
    deploy:
      replicas: 3
    ports:
      - "8002:80"

  webui:
    image: ismartio/webui:latest
    networks:
      - coinswarmnet
    deploy:
      replicas: 1
    ports:
      - "8000:80"

  worker:
    image: ismartio/worker:latest
    networks:
      - coinswarmnet
    deploy:
      replicas: 2

networks:
  coinswarmnet:
    external: true
```

Deploy with:

```bash
docker stack deploy -c docker-compose.yml dockercoins
```

### Load Balancing

- Swarm routing mesh handles traffic distribution.
- Any request to a node's published port is routed to available replicas.
- DNS-based load balancing ensures even distribution.

### Manual Scaling

```bash
docker service scale dockercoins_rng=5
docker service scale dockercoins_hasher=5
```

> ⚠️ Auto-scaling needs external tools (e.g., Prometheus + KEDA).

### HA Monitoring

```bash
docker node ls
docker service ls
docker service ps dockercoins_rng
```

Test HA by shutting down a node and verifying redistribution.

## 📊 Centralized Logging

### ELK Stack Docker Compose:

```yaml
version: '3.8'
services:
  elasticsearch:
    image: elasticsearch:8.8.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"
    networks:
      - coinswarmnet

  logstash:
    image: logstash:8.8.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    networks:
      - coinswarmnet

  kibana:
    image: kibana:8.8.0
    ports:
      - "5601:5601"
    networks:
      - coinswarmnet

networks:
  coinswarmnet:
    external: true
```

### logstash.conf:

```conf
input {
  docker {
    host => "unix:///var/run/docker.sock"
  }
}
output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
  }
}
```

Access logs via: `http://<manager-ip>:5601`

## ⚖️ System Monitoring

### Monitoring Stack Configuration

#### docker-compose-monitoring.yml:

```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - coinswarmnet
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    networks:
      - coinswarmnet
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - "8080:8080"
    networks:
      - coinswarmnet
  node_exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    networks:
      - coinswarmnet
  influxdb:
    image: influxdb:latest
    ports:
      - "8086:8086"
    networks:
      - coinswarmnet
networks:
  coinswarmnet:
    external: true
```
#### Prometheus Configuration (prometheus.yml):
```yaml
scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['master01:9090', 'worker01:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['node_exporter:9100']
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```
## 🎯 Performance Testing

| Metric                  | Before Optimization | After Scaling |
| ----------------------- | ------------------- | ------------- |
| RNG Latency (avg)       | 455ms               | 210ms         |
| Hasher Throughput       | 120 req/s           | 350 req/s     |
| Redis Hit Rate          | 78%                 | 92%           |

**Tools Used**: httping, top, vmstat, docker stats

⚠️ **In Progress**
- Nginx reverse proxy configuration
- Prometheus alert rules
- Auto-scaling logic

---

## 📈 Key Learnings

- Configured a static IP VM cluster
- Mastered Docker Swarm orchestration
- Integrated ELK Stack and Grafana
- Optimized performance via scaling

---

## ➡️ Next Steps

### Nginx Reverse Proxy:
- Create `nginx.conf` with upstream blocks
- Deploy `nginx:alpine` with mounted config
- Enable SSL termination

### Prometheus Alerts:
- Define `.rules` for CPU/memory thresholds
- Configure `alertmanager.yml` for notifications
- Test via `localhost:9090/alerts`

### Auto-Scaling:
- Integrate KEDA with Prometheus scaler
- Scale based on latency/queue depth
- Use cron/event-driven triggers

### Secure Dashboards:
- Enable HTTP Basic Auth for Grafana/Kibana
- Configure HTTPS via Let’s Encrypt
- Implement role-based access

### Automated Testing:
- Write JMeter/Locust scripts
- Push results to InfluxDB
- Build Grafana dashboards

### CI/CD:
- Create GitHub Actions workflow
- Automate image builds and deployments
- Trigger on main branch changes

---

## 📝 References

- **Course**: Big Data Foundations – Industrial University of HCMC
- **Instructors**: Mr. Huỳnh Nam & Dương Thái Bảo

**Resources**:
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/)

