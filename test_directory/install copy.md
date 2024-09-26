# Install kernel monitoring

Будем настраивать корневую машину с прометеусом и grafana и настроим получение и работу с таргетами

Будем делать по https://www.dmosk.ru/miniinstruktions.php?mini=prometheus-stack-docker#prepare
Установим docker & docker-compose

Создаем каталоги, где будем создавать наши файлы:
```
mkdir -p /opt/prometheus_stack/{prometheus,grafana,alertmanager,blackbox}
```
Создаем файл:
```
touch /opt/prometheus_stack/docker-compose.yml
cd /opt/prometheus_stack


```
в итоге имеем
```
vm11-k8s-monitoring@vm11-k8s-monitoring:/opt$ tree
.
├── containerd  [error opening dir]
└── prometheus_stack
    ├── alertmanager
    ├── blackbox
    ├── docker-compose.yml
    ├── grafana
    │   ├── alerting  [error opening dir]
    │   ├── csv  [error opening dir]
    │   ├── grafana.db
    │   ├── plugins
    │   ├── png  [error opening dir]
    │   └── provisioning
    └── prometheus
        └── prometheus.yml
11 directories, 3 files
```
создадим docker-compose.yml
```
version: '3.9'

services:

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus:/etc/prometheus/
    container_name: prometheus
    hostname: prometheus
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    ports:
      - 9090:9090
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default

  node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    container_name: exporter
    hostname: exporter
    command:
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
      - --collector.filesystem.ignored-mount-points
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
    ports:
      - 9100:9100
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
  grafana:
    image: grafana/grafana
    user: root
    depends_on:
      - prometheus
    ports:
      - 3000:3000
    volumes:
      - ./grafana:/var/lib/grafana
      - ./grafana/provisioning/:/etc/grafana/provisioning
    container_name: grafana
    hostname: grafana
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default


networks:
  default:
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16

```

prometheus.yml содержит
```
scrape_configs:
  - job_name: node
    scrape_interval: 5s
    static_configs:
    - targets: ['node-exporter:9100']
  - job_name: test_deploy_4
    scrape_interval: 5s
    static_configs:
    - targets: ['192.168.1.81:9100']

  - job_name: loadbalancers
    scrape_interval: 5s
    static_configs:
    - targets: ['192.168.1.74:8404']

```
Тут у нас уже имеется ТРИ node_exporter таргета, один test_deploy4, loadbalancers, и базовая машина.

поднимаем контейнеры по docker-compose up -d

