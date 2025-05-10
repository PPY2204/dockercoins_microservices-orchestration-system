🐳 Docker Swarm Microservices System – Monitoring & Orchestration
Deploy, monitor, and optimize software system built on microservices architectures using Docker Swarm Orchestration
* Project Overview
- Status: Ongoing (~80% complete)
- Team Size: 2 members
- My Role: Main – Responsible for infrastructure setup, orchestration, and deployment
* Technology Stack
- Orchestration & Containerization: Docker, Docker Swarm
- Monitoring & Logging:
    + ELK Stack (Elasticsearch, Logstash, Kibana)
    + Prometheus + InfluxDB + Grafana
- Others: Linux (Ubuntu), VirtualBox, Nginx (in progress)

* System Setup (Virtual Environment)
1. Virtualized Environment Setup
- Installed essential packages:
  sudo apt update
  sudo apt install ifupdown net-tools openssh-client openssh-server
- Created 10 Ubuntu virtual machines using VirtualBox:
    + 3 Manager Nodes: master01 (192.168.100.10), master02 (192.168.100.11), master03 (192.168.100.12)
    + 7 Worker Nodes: worker01 → worker07 (192.168.100.13 → 192.168.100.19)
- Each VM configured with:
    + NAT Adapter: for internet access (default gateway: 10.0.2.2)
    + Host-only Adapter:
        o Internal VM communication using subnet 192.168.100.0/24
        o Host-to-VM SSH access supported
- Manually assigned static IPs via /etc/network/interfaces
    #interfaces (5) file used by ifup (8) and ifdown (8)
    #Include files from /etc/network/interfaces.d: source /etc/network/interfaces.d/*
    auto lo
    iface lo inet loopback
    auto enp0s3 iface enp0s3 inet dhcp
    auto enp0s8
    iface enp0s8 inet static address 192.168.100.10
  Ctrl O
  Enter
  Ctrl X
- Then sudo reboot and validated using "ping", "ip a", "route -n", "ssh", "ifconfig -a"
- Check NAT network:
  ping www.google.com → see network reply successfully
- Check Host-only network:
    + From windows machine, open cmd, then type ping 192.168.100.10
    + Or try ssh connection from windows machine to master01 machine
      Start ssh server in master01 machine and check the status is active (running)
      From external windows machine, we use cmd to ssh osboxes@192.168.100.10
(*) Do the same for 9 machines master02, master03, worker01, worker02, worker03, worker04, worker05, worker06, worker07.
The IP part of the host-only network corresponds to 192.168.100.11, 192.168.100.12, 192.168.100.13, 192.168.100.14, 192.168.100.15, 192.168.100.16, 192.168.100.17, 192.168.100.18, 192.168.100.19
=> If all test cases are successful, the construction of the virtual network infrastructure for 10 computers (computer group) is completed.

- Static IPs were assigned for swarm initialization (--advertise-addr).
2. Docker Swarm Deployment
- Swarm cluster initialized and nodes joined via generated token.
- Overlay network created (coinswarmnet) to allow communication between services across nodes.
3. Services Deployed
- Initial microservices provided by instructor:
redis, rng, hasher, worker, webui
- Deployed as Docker services on swarm with correct port mapping and scaling:

PHẦN 2: KIỂM TRA CÁC GÓI PHẦN MỀM TRONG HỆ THỐNG VỚI DOCKER ENGINE
