# setup domain di cloudflare
pergi ke situs cloudflare > pilih akun > website > klik
domain yang tersedia
![](.1deploy_waysgallery_menggunakan_docker_images/9a1bdc5a.png)

pilih dns > add record > masukan domain dan ip public
dari gateway
![](.1deploy_waysgallery_menggunakan_docker_images/2d5885b8.png)
![](.1deploy_waysgallery_menggunakan_docker_images/6189e068.png)

# setup nginx on top docker

install docker di gateway menggunakan script
```shell
#!/bin/bash

read -p "ketik y untuk menginstall docker, ketik sembarang kata untuk membatalkan   " choice

if [ $choice = "y" ]
then
    echo "###########################"
    echo "update repository"
    echo "###########################"
    sudo apt-get update -y

    echo "menghapus semua versi lama docker bila ada"
    sudo apt-get remove docker docker-engine docker.io containerd runc -y
    
    echo "###########################"
    echo "install depedency yang diperlukan"
    echo "###########################"
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release -y
    
    echo "###########################"
    echo "install GPG key"
    echo "###########################"
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    echo "###########################"
    echo "set up repository"
    echo "###########################"
    echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    echo "###########################"
    echo "update repository lagi"
    echo "###########################"
    sudo apt-get update -y
    
    echo "###########################"
    echo "install docker engine, containerd, dan docker compose"
    echo "###########################"
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y

    echo "###########################"
    echo "tambahkan user yang ada sekarang ke dalam grup docker"
    echo "###########################"
    sudo usermod -aG docker $(whoami)
    pwd
    
    echo "###########################"
    echo "docker berhasil di install"
    echo "###########################"
    exec bash
else
    echo "script berhenti"
fi
```
![](.1deploy_waysgallery_menggunakan_docker_images/28ad355f.png)
![](.1deploy_waysgallery_menggunakan_docker_images/68b8b655.png)

buat file yang berisi default konfigurasi nginx.conf karena akan 
di bind mount
```shell

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
![](.1deploy_waysgallery_menggunakan_docker_images/d8328439.png)

buat file docker compose
```shell
version: '3.7'

services:
  nginx:
    container_name: nginx
    image: nginx:latest
    restart: unless-stopped
    stdin_open: true
    ports:
      - 80:80
      - 443:443
    volumes:
      - /home/kelompok1/konfigurasi_nginx/sites-enabled:/etc/nginx/sites-enabled
      - /home/kelompok1/konfigurasi_nginx/nginx.conf:/etc/nginx/nginx.conf
      - /home/kelompok1/konfigurasi_nginx/letsencrypt:/etc/letsencrypt
```
![](.1deploy_waysgallery_menggunakan_docker_images/632dddbe.png)

jalankan docker compose 
```shell
docker compose up -d
```
![](.1deploy_waysgallery_menggunakan_docker_images/71fa94b8.png)

# setup reverse proxy

buat file untuk konfigurasi reverse proxy untuk frontend dan backend
di folder yang sudah di bind mount menggunakan sudo,
gunakan ip private untuk proxy pass

```shell
server {
    server_name literature.studentdumbways.my.id;

    location / {
         proxy_pass http://10.36.116.163:3000;
    }
}

server {
    server_name api.literature.studentdumbways.my.id;

    location / {
         proxy_pass http://10.36.116.163:5000;
    }
}
```
![](.1deploy_waysgallery_menggunakan_docker_images/4698074c.png)

buka file nginx.conf yang sudah di mount menggunakan sudo, tambahkan folder konfigurasi
agar dapat dibaca (nginx di docker tidak menginclude sites-enabled)
```shell
include /etc/nginx/sites-enabled/*.conf;
```
![](.1deploy_waysgallery_menggunakan_docker_images/73e4ee78.png)

masuk ke container
```shell
docker container exec -i -t nama_container /bin/bash
```
![](.1deploy_waysgallery_menggunakan_docker_images/b876489f.png)

cek syntax nginx
```shell
nginx -t
```
![](.1deploy_waysgallery_menggunakan_docker_images/b69cc40d.png)

reload nginx
```shell
service nginx reload
```
![](.1deploy_waysgallery_menggunakan_docker_images/0b8a0cde.png)

# pemasangan https menggunakan certbot

masuk ke container
```shell
docker container exec -i -t nama_container /bin/bash
```
![](.1deploy_waysgallery_menggunakan_docker_images/9453a393.png)

update package
```shell
apt update
```
![](.1deploy_waysgallery_menggunakan_docker_images/71a91a1a.png)

install depdensi yang diperlukan
```shell
apt install python3 python3-venv libaugeas0 -y
```
![](.1deploy_waysgallery_menggunakan_docker_images/9171d64d.png)

buat virtualenv
```shell
python3 -m venv /opt/certbot/
```
![](.1deploy_waysgallery_menggunakan_docker_images/244579cf.png)
```shell
/opt/certbot/bin/pip install --upgrade pip
```
![](.1deploy_waysgallery_menggunakan_docker_images/1efe048c.png)

install cerbot untuk nginx
```shell
/opt/certbot/bin/pip install certbot certbot-nginx
```
![](.1deploy_waysgallery_menggunakan_docker_images/9efcef47.png)

tambahkan path untuk memastikan cerbot dapat berjalan
```shell
ln -s /opt/certbot/bin/certbot /usr/bin/certbot
```
![](.1deploy_waysgallery_menggunakan_docker_images/449bb547.png)

jalankan certbot untuk nginx
```shell
certbot --nginx
```

masukan email

![](.1deploy_waysgallery_menggunakan_docker_images/8b8749fc.png)

input yes

![](.1deploy_waysgallery_menggunakan_docker_images/b1872a8e.png)

input N
![](.1deploy_waysgallery_menggunakan_docker_images/ee8498f4.png)

tekan enter untuk memasukan semua domain
![](.1deploy_waysgallery_menggunakan_docker_images/616287d5.png)

# setup database
buat docker compse untuk mysql
```shell
version: '3.7'

services:
  database:
    container_name: database
    image: mysql:latest
    environment:
      MYSQL_DATABASE: 'literature'
      MYSQL_USER: 'kelompok1'
      MYSQL_PASSWORD: 'kelompok1'
      MYSQL_ROOT_PASSWORD: 'kelompok1'
    ports:
      - '3306:3306'
    restart: unless-stopped
    volumes:
      - ~/konfigurasi_mysql:/var/lib/mysql
```
![](.1deploy_waysgallery_menggunakan_docker_images/a9255d05.png)

jalankan docker compose
```shell
docker compose up -d
```
![](.1deploy_waysgallery_menggunakan_docker_images/95286f6e.png)

# setup frontend

clone literature frontend
```shell
git clone https://github.com/dumbwaysdev/literature-frontend
```
![](.1deploy_waysgallery_menggunakan_docker_images/0df3d38f.png)

buat file .dockerignore, masukan file & folder mana saja yang tidak
ingin dimasukan ketika di build
```shell
nano .dockerignore
```
![](.1deploy_waysgallery_menggunakan_docker_images/79ead10f.png)

ubah file src/config/config.js, ubah domain
![](.1deploy_waysgallery_menggunakan_docker_images/38e2808f.png)

buat Dockerfile
```shell
FROM node:14-alpine as build
WORKDIR /home/app
COPY . .
RUN npm install

FROM node:14-alpine
WORKDIR /home/app
COPY --from=build /home/app /home/app
EXPOSE 3000
CMD ["npm","start"]
```
![](.1deploy_waysgallery_menggunakan_docker_images/119a1bce.png)

build dockerfile
```shell
docker build -t reiya24/literature-frontend -t naninanides/literature-frontend .
```
![](.1deploy_waysgallery_menggunakan_docker_images/0e3696e5.png)

buat docker compose
```shell
version: '3.7'
services:
 frontend:
   container_name: frontend
   image: naninanides/literature-frontend
   stdin_open: true
   restart: unless-stopped
   ports:
    - 3000:3000
```
jalankan docker compose 
```shell
docker compose up -d
```
![](.1deploy_waysgallery_menggunakan_docker_images/91b27aae.png)

# setup backend

clone literature backend
```shell
git clone https://github.com/dumbwaysdev/literature-backend
```
![](.1deploy_waysgallery_menggunakan_docker_images/578c6b71.png)

ubah konfigurasi file config/config.json

![](.1deploy_waysgallery_menggunakan_docker_images/a3cc5f28.png)

buat Dockerfile

```shell
FROM node:14-alpine AS build
WORKDIR /home/app
COPY .. .
RUN npm install
RUN npm install -g sequelize-cli
RUN npx sequelize db:migrate

FROM node:14-alpine
COPY --from=build /home/app /home/app
WORKDIR /home/app
CMD ["node","server.js"]
```
![](.1deploy_waysgallery_menggunakan_docker_images/820aa7aa.png)

build Dockerfile
```shell
docker build -t reiya24/literature-backend -t naninanides/literature-backend .
```
![](.1deploy_waysgallery_menggunakan_docker_images/1ad0df87.png)

buat docker compose

```shell
version: '3.7'
services:
 backend:
  image: reiya24/literature-backend
  container_name: backend
  stdin_open: true
  restart: unless-stopped
  ports:
   - 5000:5000
```
![](.1deploy_waysgallery_menggunakan_docker_images/5f95409b.png)

jalankan docker compose
![](.1deploy_waysgallery_menggunakan_docker_images/90ab2177.png)

# membuat private registry
buat docker compose
```shell
version: '3.3'
services:
  registry:
    container_name: private_registry
    image: registry:latest
    restart: always
    ports:
    - "5005:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - /home/kelompok1/private_registry/data:/datae
```
jalankan docker compose
```shell
docker compose up -d
```
![](.1deploy_waysgallery_menggunakan_docker_images/4fea7f68.png)

tambahkan file /etc/docker/daemon.json, masukan ip kita
```shell
{
   "insecure-registries" : ["10.36.116.163:5005"]
}
```
![](.1deploy_waysgallery_menggunakan_docker_images/43fcc24c.png)

ubah tag image
![](.1deploy_waysgallery_menggunakan_docker_images/e295c18d.png)

push imagenya
![](.1deploy_waysgallery_menggunakan_docker_images/87ffab2f.png)
# hasil website

![](.1deploy_waysgallery_menggunakan_docker_images/ab6898d1.png)

![](.1deploy_waysgallery_menggunakan_docker_images/1d289b69.png)

![](.1deploy_waysgallery_menggunakan_docker_images/2dcc087b.png)