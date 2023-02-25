# instalasi node exporter on top docker

buat file docker compose
```yaml
version: '3.7'
services:
  node_exporter:
    container_name: node_exporter
    image: bitnami/node-exporter:latest 
    stdin_open: true
    restart: unless-stopped
    ports:
      - 9100:9100
```

buat ansible playbook
```yaml
---
- hosts: appserver,gateway
  become: true
  gather_facts: true
  tasks:
    - name: buat folder compose_file
      file:
        path: /home/{{ansible_user}}/compose_file
        state: directory
        owner: "{{ansible_user}}"

    - name: copy node exporter compose files
      copy:
        src: compose_file/node_exporter_compose.yaml
        dest: /home/{{ansible_user}}/compose_file/node_exporter_compose.yaml
        owner: "{{ansible_user}}"

    - name: deploy Docker Compose
      community.docker.docker_compose:
        project_src: /home/{{ansible_user}}/compose_file
        files:
          - node_exporter_compose.yaml
```

jalankan ansible playbook
```yaml
ansible-playbook 4install_node_exporter.yaml
```
![](.3monitoring_images/2113e62a.png)

# instalasi prometheus, copy prometheus.yml dan grafana

buat file prometheus.yml
```shell
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ["https://gateway-node.reiya.my.id","https://appserver-node.reiya.my.id","https://monitoring-node.reiya.my.id"]
```
![](.3monitoring_images/de91b379.png)

buat docker compose
```yaml
version: '3.7'
services:
  prometheus:
    container_name: prometheus
    image: bitnami/prometheus:latest
    stdin_open: true
    restart: unless-stopped
    ports:
      - 9090:9090
    volumes:
      - ~/konfigurasi_prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
  grafana:
    container_name: grafana
    image: grafana/grafana:latest
    stdin_open: true
    restart: unless-stopped
    ports:
      - 3123:3000
    volumes:
      - grafana-data:/etc/grafana/provisioning
      - grafana-data:/var/lib/grafana

volumes:
  grafana-data:
```

buat ansible playbook
```shell
---
- hosts: monitoring
  become: true
  gather_facts: true
  tasks:
    - name: buat folder konfigurasi_prometheus
      file:
        path: /home/{{ansible_user}}/konfigurasi_prometheus
        state: directory
        owner: "{{ansible_user}}"

    - name: copy file prometheus.yaml
      copy:
        src: prometheus/prometheus.yml
        dest: /home/{{ansible_user}}/konfigurasi_prometheus/prometheus.yml
        owner: "{{ansible_user}}"

    - name: copy prometheus_grafana_compose.yaml
      copy:
        src: compose_file/prometheus_grafana_compose.yaml
        dest: /home/{{ansible_user}}/compose_file/prometheus_grafana_compose.yaml
        owner: "{{ansible_user}}"

    - name: deploy Docker Compose
      community.docker.docker_compose:
        project_src: /home/{{ansible_user}}/compose_file
        files:
          - prometheus_grafana_compose.yaml
        remove_orphans: true
```

jalankan ansible playbook
```shell
ansible-playbook 5install_prometheus_grafana.yaml
```
![](.3monitoring_images/90fd13e6.png)

# instalasi nginx dan reverse proxy,
buat file reverse proxy
```shell


server {
    server_name reiya.my.id;

    location / {
         proxy_pass http://10.116.106.88:3000;
    }
}


server {
    server_name appserver-node.reiya.my.id;

    location / {
         proxy_pass http://10.116.106.88:9100;
    }
}




server {
    server_name gateway-node.reiya.my.id;

    location / {
         proxy_pass http://10.116.106.56:9100;
    }
}




server {
    server_name prom.reiya.my.id;

    location / {
         proxy_pass http://10.116.106.36:9090;
    }
}

server {
    server_name dashboard.reiya.my.id;

    location / {
         proxy_pass http://10.116.106.36:3123;
    }
}
```

ansible playbook
```shell
---
- hosts: gateway
  become: true
  gather_facts: true
  tasks:
    - name: "install nginx menggunakan apt"
      apt:
        name: nginx
        state: latest
        update_cache: true
    - name: "jalankan service nginx"
      service:
        name: nginx
        state: started
    - name: "copy file konfigurasi nginx"
      copy:
        src: sites-enabled/
        dest: /etc/nginx/sites-enabled
    - name: "restart service nginx"
      service:
        name: nginx
        state: reloaded
    - name: Install certbot
      pip:
        name:
          - certbot
          - certbot-nginx
        executable: pip3
      tags:
        - nginx
        - certbot
    - name: jalankan certbot
      shell:
        "sudo certbot --nginx --non-interactive --agree-tos -d reiya.my.id -d appserver-node.reiya.my.id -d  gateway-node.reiya.my.id -d prom.reiya.my.id -d dashboard.reiya.my.id  --email reiya2307@gmail.com"



```

jalankan ansible playbook

```shell
ansible-playbook 6install_nginx_reverse_proxy.yaml
```
![](.3monitoring_images/ca4c52fc.png)

# setup grafana

masuk ke dashboard grafana, masukan user admin dan password admin
![](.3monitoring_images/903645fe.png)

masukan password baru
![](.3monitoring_images/79fd548e.png)

## menambahkan data source

pilih icon setting > data source

add data source
![](.3monitoring_images/aa82f3ba.png)

pilih prometheus
![](.3monitoring_images/11d2b9cf.png)

masuk domain prometheus
![](.3monitoring_images/51e56102.png)

masukan scrape interval dan query timeout
![](.3monitoring_images/7e021342.png)

klik save and test
![](.3monitoring_images/e69e1999.png)

## membuat dashboard
pada icon kotak empat, klik new dashboard
![](.3monitoring_images/1e8ca41a.png)

add a new panel
![](.3monitoring_images/fa9402d3.png)

### membuat instance untuk menampilkan persentase CPU
masukan query, masukan title
```shell
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle", instance="10.116.106.88:9100"}[5m])) * 100 ) 
```
![](.3monitoring_images/c3319cbf.png)
lakukan hal yang sama untuk gateway

### membuat instance untuk menampilkan persentase memory
masukan query
```shell
100 * ((node_memory_MemTotal_bytes{instance="10.116.106.88:9100"} - node_memory_MemFree_bytes{instance="10.116.106.88:9100"} - node_memory_Buffers_bytes{instance="10.116.106.88:9100"} - node_memory_Cached_bytes{instance="10.116.106.88:9100"}) / node_memory_MemTotal_bytes{instance="10.116.106.88:9100"})
```
![](.3monitoring_images/449966a7.png)
lakukan hal yang sama untuk gateway

save dashboard dengan mengklik icon save
![](.3monitoring_images/dbb2453d.png)

masukan nama dashboard

![](.3monitoring_images/34cd5b7d.png)

# menggunakan template

template dari
https://grafana.com/grafana/dashboards/1860-node-exporter-full/

pada menu dashboard, plilih import
![](.3monitoring_images/b6796c53.png)

masukan kode
![](.3monitoring_images/667d4e73.png)

masukan form
![](.3monitoring_images/17b212a5.png)

masukan nama dan data source
![](.3monitoring_images/ad1dad49.png)

![](.3monitoring_images/9aacd6f0.png)

save dashboard
![](.3monitoring_images/399d6400.png)

# alerting

## setup integrasi slack dan grafana
masuk ke https://api.slack.com , pilih create ann app
![](.3monitoring_images/76cffbf8.png)

pilih from strach
![](.3monitoring_images/94e239a8.png)

masukan app name dan workspace slack
![](.3monitoring_images/d5d51234.png)

pilih incoming webhook
![](.3monitoring_images/fa89d412.png)

activate incoming webhook
![](.3monitoring_images/6410b77d.png)

add new webhook to workspace
![](.3monitoring_images/193f8d4d.png)

pilih channel
![](.3monitoring_images/9a7d0a6b.png)

copy webhook url
![](.3monitoring_images/f858eea9.png)

pada dashboard grafana, klik alerting, notification policies
![](.3monitoring_images/1b6f395f.png)

pada root policies, pilih edit > create a contact point
masukan nama, contact point types, webhook url, klik test
![](.3monitoring_images/e29d46b9.png)

send test notification
![](.3monitoring_images/c73ee496.png)

pengiriman notifikasi berhasil
![](.3monitoring_images/49c13af7.png)

save contact point
![](.3monitoring_images/a1c369b3.png)