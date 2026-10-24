```yml
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    networks:
      - prometheus_net
    container_name: prometheus

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    restart: unless-stopped
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning/:/etc/grafana/provisioning/
      - ./grafana.ini:/etc/grafana/grafana.ini
    networks:
      - prometheus_net
    container_name: grafana

  pushgateway:
    image: prom/pushgateway:latest
    container_name: pushgateway
    restart: unless-stopped
    ports:
      - "9091:9091"
    networks:
      - prometheus_net

  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    ports:
      - "9100:9100"
    command:
      - '--path.rootfs=/host'
    network_mode: host
    pid: host
    volumes:
      - '/:/host:ro,rslave'
    deploy:
      mode: global
      resources:
        limits:
          cpus: "0.1"
          memory: 1024M
        reservations:
          cpus: "0.05"
          memory: 128M

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor-exporter
    privileged: true
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    devices:
      - /dev/kmsg
    deploy:
      mode: global
      resources:
        limits:
          cpus: "0.2"
          memory: 512M
        reservations:
          cpus: "0.05"
          memory: 128M

networks:
  prometheus_net:

volumes:
  prometheus_data:
  grafana_data:
```


> [!info] Monitoring Stack Overview
> **Components:** Grafana, Prometheus, cAdvisor, Node Exporter, and Pushgateway.
> 
> **Deployment Note:** This configuration is natively designed for Docker Swarm (`docker stack`). To deploy it as a standalone `docker-compose` setup on individual servers, you must remove the `mode: global` directive from the `cAdvisor` and `node_exporter` services.