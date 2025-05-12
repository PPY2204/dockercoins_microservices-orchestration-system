# 🐳 Docker Swarm Microservices System – Monitoring & Orchestration

![Docker Swarm](https://img.shields.io/badge/Docker-Swarm-2496ED?logo=docker)
![Status](https://img.shields.io/badge/Status-80%25%20Complete-yellowgreen)
![Team](https://img.shields.io/badge/Team-2%20Members-blue)

This project involves deploying, monitoring, and optimizing a microservices-based distributed system using **Docker Swarm orchestration**. It simulates a real-world scalable environment with centralized logging and system health monitoring.

## 📌 Project Overview

A production-like microservices system demonstrating:

- **Container orchestration** with Docker Swarm
- **Centralized monitoring** with Prometheus/Grafana
- **Distributed logging** using ELK Stack
- **Automated scaling** of stateless services

## 🚀 Technology Stack

### Core Components
| Component       | Purpose                          | Version     |
|----------------|----------------------------------|-------------|
| Docker Swarm   | Container orchestration          | 24.10       |
| Ubuntu Server  | Host OS                          | 24.10       |

### Monitoring Stack
| Tool           | Function                         | Port        |
|----------------|----------------------------------|-------------|
| Prometheus     | Metrics collection               | 9090        |
| Grafana        | Visualization                    | 3000        |
| cAdvisor       | Container metrics                | 8080        |
| node_exporter  | Host metrics                     | 9100        |
| InfluxDB       | Time-series DB for metrics       | 8086        |

### Logging Stack
| Component      | Function                         | Port        |
|----------------|----------------------------------|-------------|
| Elasticsearch  | Log storage                      | 9200        |
| Logstash       | Log processing                   | 5044        |
| Kibana         | Log visualization                | 5601        |

## 🛠️ System Setup (Virtual Environment)

### 1. Virtual Machine Setup
- Created 10 Ubuntu virtual machines using VirtualBox:
  - **3 Manager Nodes**: master01 (192.168.100.10), master02 (192.168.100.11), master03 (192.168.100.12)
  - **7 Worker Nodes**: worker01 → worker07 (192.168.100.13 → 192.168.100.19)

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
Each VM was configured with two network adapters:
- **NAT Adapter**:
  - Internet access (Gateway: `10.0.2.2`)
- **Host-only Adapter**:
  - Internal VM communication using subnet `192.168.100.0/24`
  - Host-to-VM SSH access supported

#### Static IP Configuration
Manually assigned static IPs via /etc/network/interfaces
```bash
#interfaces (5) file used by ifup (8) and ifdown (8)
#Include files from /etc/network/interfaces.d: source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback
auto enp0s3
iface enp0s3 inet dhcp
auto enp0s8
iface enp0s8 inet static
address 192.168.100.10
```

#### Validation Commands
```bash
ping www.google.com          # Test internet
ping 192.168.100.11          # Test internal network
ssh osboxes@192.168.100.10    # Test SSH connectivity from external windows machine
```



## ⚙️ System Description

### 1. Docker Swarm Deployment
```bash
# On master01:
docker swarm init --advertise-addr 192.168.100.10

# On other nodes:
docker swarm join --token <token> 192.168.100.10:2377
```

#### Overlay Network
```bash
sudo docker network create --scope=swarm --attachable -d overlay --opt encrypted coinswarmnet
```

### 2. Services Deployed
| Service  | Image                    | Replicas | Ports       | Network       |
|----------|--------------------------|----------|-------------|---------------|
| redis    | redis:alpine            | 3        | 6379:6379   | coinswarmnet  |
| webui    | dockersamples/visualizer| 1        | 8080:8080   | coinswarmnet  |

| redis    | redis:alpine            | 3        | 6379:6379   | coinswarmnet  |
| rng      | rng                     | 2        | 8001:80     | coinswarmnet  |
| webui    | webui                   | 1        | 8080:8080   | coinswarmnet  |
#### Deployment Example
```bash
docker service create \
    --name rng \
    --network coinswarmnet \
    --replicas 2 \
    --publish published=8001,target=80 \
    custom-rng:latest
```

### 3. Centralized Logging
- Set up **ELK Stack** (Elasticsearch, Logstash, Kibana)
- Deployed via Docker Compose and routed logs through Logstash:
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
- Logs visualized at `http://<manager-ip>:5601`

### 4. System Monitoring
- Installed Prometheus, cAdvisor, node_exporter
- Metrics stored in InfluxDB and visualized in Grafana:
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['master01:9090', 'worker01:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['node_exporter:9100']
```

### 5. Performance Testing
| Metric               | Before Optimization | After Scaling |
|----------------------|---------------------|---------------|
| RNG Latency (avg)    | 455ms               | 210ms         |
| Hasher Throughput    | 120 req/s           | 350 req/s     |
| Redis Hit Rate       | 78%                 | 92%           |

- Used `httping` for service latency
- Used `top`, `vmstat`, `docker stats` for system metrics

### 6. (In Progress)
- Configuring **Nginx** as a reverse proxy for routing and load balancing
- Adding alerting rules and auto-scaling logic

## 📈 Key Learnings & Contributions

- Built full virtualized infrastructure with static IPs
- Gained hands-on experience with Docker Swarm orchestration and networking
- Set up centralized logging with ELK Stack and visual dashboards with Grafana
- Diagnosed and optimized performance issues using metrics and logs

## ➡️ Next Steps

- [ ] Finalize Nginx reverse proxy configuration
- [ ] Implement automated scaling policies
- [ ] Add Prometheus alert rules
- [ ] Document recovery and restart procedures

## 📝 References
- Course: *Big Data Foundations – Industrial University of HCMC*  
- Instructors: Thầy Huỳnh Nam & Dương Thái Bảo
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/naming/)
