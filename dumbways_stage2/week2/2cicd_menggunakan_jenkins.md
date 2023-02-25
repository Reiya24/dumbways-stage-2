# install jenkins on top docker

buat file docker compose
```shell
version: '3.7'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins:latest-jdk11
    privileged: true
    user: root
    restart: unless-stopped
    ports:
      - 8080:8080
    volumes:
      - ~/jenkins_config:/var/jenkins_home
      - ~/jenkins-docker-certs:/certs/clien
```

jalankan docker compose
```shell
docker compose up -d
```
![](.2cicd_menggunakan_jenkins/4d1518ae.png)

# buat reverse proxy untuk jenkins

tambahkan domain di cloudflare
![](.2cicd_menggunakan_jenkins/fc409e1a.png)

buat file konfigurasi reverse proxy di webserver
```shell
server {
    server_name jenkins.literature.studentdumbways.my.id;

    location / {
             proxy_pass http://10.36.116.151:8080;
    }
}
```
![](.2cicd_menggunakan_jenkins/5d84e6d4.png)

masuk ke container nginx
```shell
docker container exec -it nginx bash
```
![](.2cicd_menggunakan_jenkins/16242417.png)

test syntax nginx
```shell
nginx -t
```
![](.2cicd_menggunakan_jenkins/95c11b83.png)

reload nginx
```shell
service nginx reload
```
![](.2cicd_menggunakan_jenkins/99e86433.png)

jalankan certbot untuk mendapatkan SSL
```shell
certbot --nginx
```
![](.2cicd_menggunakan_jenkins/78a264ef.png)

# setup jenkins
cek logs container jenkins untuk mendapatkan password
```shell
docker container logs jenkins
```
![](.2cicd_menggunakan_jenkins/12dac5fc.png)

copy kode tersebut, dan paste di web browser jenkins
![](.2cicd_menggunakan_jenkins/3a296ebc.png)

pilih select plugin to install
![](.2cicd_menggunakan_jenkins/5855a645.png)

cari ssh agent dan ceklis
![](.2cicd_menggunakan_jenkins/94a73111.png)

tunggu proses instalasi
![](.2cicd_menggunakan_jenkins/554cfd96.png)

masukan form yang diperlukan
![](.2cicd_menggunakan_jenkins/0b930d8e.png)

setup domain jenkins
![](.2cicd_menggunakan_jenkins/6a21daeb.png)

pilih start using jenkins
![](.2cicd_menggunakan_jenkins/b530d4f2.png)

# setup SSH credentials

generate ssh
![](.2cicd_menggunakan_jenkins/4d60cc76.png)

copy privete keynya untuk dipaste ke dalam jenkins
![](.2cicd_menggunakan_jenkins/4f28424f.png)

pada website jenkins pilih manage jenkins
![](.2cicd_menggunakan_jenkins/9df5867d.png)

configure global security
![](.2cicd_menggunakan_jenkins/28e2889e.png)

pilih global
![](.2cicd_menggunakan_jenkins/0b6b1dd0.png)

pilih add credentials
![](.2cicd_menggunakan_jenkins/d548e072.png)

masukan form
![](.2cicd_menggunakan_jenkins/b0e76a17.png)

# tambahkan public key di github
copy publik key
![](.2cicd_menggunakan_jenkins/aecab9b1.png)

paste di github, masuk ke website github, settings
![](.2cicd_menggunakan_jenkins/d6806aa2.png)

SSH and GPG keys
![](.2cicd_menggunakan_jenkins/525b7fec.png)

new SSH keys
![](.2cicd_menggunakan_jenkins/d9664f81.png)

paste isi dari public key
![](.2cicd_menggunakan_jenkins/f724949a.png)

# tambahkan public key di appserver
copy publik key
![](.2cicd_menggunakan_jenkins/aecab9b1.png)

paste di authorized_keys di appserver
![](.2cicd_menggunakan_jenkins/423013ef.png)

# seting agar bisa terkoneksi dengan github di koneksi pertama
pilih manage jenkins > configure global security
![](.2cicd_menggunakan_jenkins/fde21466.png)

scroll ke bagian paling bawah, accept first connection
![](.2cicd_menggunakan_jenkins/5b85c366.png)

# setup notifikasi dengan slack
install plugin slack notification, dashboard > manage jenkins, manage plugins
![](.2cicd_menggunakan_jenkins/6a050e34.png)

pada available plugins, cari plugin slack notification, pilih
download now and isntall after restart
![](.2cicd_menggunakan_jenkins/2fe725e8.png)

tunggu proses instalasi
![](.2cicd_menggunakan_jenkins/6f310086.png)
![](.2cicd_menggunakan_jenkins/98d3639b.png)

buka halaman https://my.slack.com/services/new/jenkins-ci, lalu pilih workspace
![](.2cicd_menggunakan_jenkins/c176319e.png)

pilih channel yang akan digunakan
![](.2cicd_menggunakan_jenkins/490bb934.png)

setelah itu scroll kebawah, kita akan menemukan team subdomain, dan integration token credential ID
![](.2cicd_menggunakan_jenkins/281f245f.png)

copy integration token credential ID, lalu tambahkan di credential jenkins, pada menu kind, pilih secret text
![](.2cicd_menggunakan_jenkins/053bd424.png)

setelah itu, pada dashboard jenkins > manage jenkins > 
configure system, cari menu slack, masukan form yang diperlukan,
untuk workspace masukan team subdomain tadi, disarankan untuk 
test connection terlebih dahulu sebelum save
![](.2cicd_menggunakan_jenkins/9c2f79c7.png)

integrasi berhasil
![](.2cicd_menggunakan_jenkins/3d371c84.png)

# membuat private repository frontend

buat repository baru di github
![](.2cicd_menggunakan_jenkins/1dd19945.png)

masukan form
![](.2cicd_menggunakan_jenkins/5f1a5b44.png)

pada folder frontend di appserver, inisialisasi git
```shell
git init
```
![](.2cicd_menggunakan_jenkins/46e27379.png)

tambahkan email dan nama di git config
```shell
git config --global user.email "email"
git config --global user.name "password"
```
![](.2cicd_menggunakan_jenkins/388e232e.png)

masukan semua file ke dalam staging area
```shell
git add .
```
![](.2cicd_menggunakan_jenkins/18d9772d.png)

lakukan commit
```shell
git commit -m "pertama"
```
![](.2cicd_menggunakan_jenkins/82471e11.png)

buat branh main
```shell
branch -M
```
![](.2cicd_menggunakan_jenkins/ccaf7548.png)

tambahkan git remote ke repository yang kita buat
```shell
git remote add origin git@github.com:Reiya24/literature-frontend.git
```
![](.2cicd_menggunakan_jenkins/64ce2393.png)

lakukan push
```shell
git push -u origin main
```
![](.2cicd_menggunakan_jenkins/d5bab992.png)

# setup github webhook
pada project github, pilih settings, webhook
![](.2cicd_menggunakan_jenkins/901bec4b.png)

pilih add webhooks
![](.2cicd_menggunakan_jenkins/4b86507f.png)

untuk payloadmasukan url jenkins dan tambahkan github-webhook, 
content type pilih
applicatoin/jsonklik send me
everything
![](.2cicd_menggunakan_jenkins/d90918ca.png)

setup berhasil
![](.2cicd_menggunakan_jenkins/6cacfab6.png)


# membuat pipeline job

pilih create a job
![](.2cicd_menggunakan_jenkins/b6a7f9a7.png)

masukan nama job,lalu pilih pipeline
![](.2cicd_menggunakan_jenkins/6f65456b.png)

pada build trigger, pilih GitHub hook trigger for GITScm polling
![](.2cicd_menggunakan_jenkins/fe4861e4.png)

pada pipeline, pilih pipeline from SCM, masukan repository URL dan
credentials, pilih branch, lalu masukan path dari jenkinsfile,
matikan lightweight checkout
![](.2cicd_menggunakan_jenkins/7197bdff.png)

# pipeline
pada folder frontend, buat script pipeline bernama Jenkinsfile
```shell
pipeline {
    agent any

    environment{
        def branch = "main"
        def nama_repository = "origin"
        def directory = "~/literature-frontend"
        def credential = 'appserver'
        def server = 'kelompok1@10.36.116.163'
        def docker_image = 'reiya24/literature-frontend'
        def nama_container = 'frontend'
    }

    options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {

        stage('kirim notifikasi ke slack') {
            steps {
                slackSend(message: "mulai job baru : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
        }

        stage('pull repository dari github ') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    git pull ${nama_repository} ${branch}
                    exit
                    EOF"""
                }
            }
        }

        stage('docker compose down') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker compose down
                    exit
                    EOF"""
                }
            }
        }

        stage('build image frontend') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker build -t ${docker_image}:latest .
                    exit
                    EOF"""
                }
            }
        }

        stage('jalankan docker compose') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker compose up -d
                    exit
                    EOF
                    """
                }
            }
        }
        
        stage('push image ke dockerhub') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker image push ${docker_image}:latest
                    exit
                    EOF"""
                }
            }
        }
    }

    post {

        aborted {
            slackSend(message: "build digagalkan secara manual : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            slackSend(message: "build failed lmao : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        success {
            slackSend(message: "build success horeeeee : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        
    }
}
```
![](.2cicd_menggunakan_jenkins/40505255.png)

lakukan git push
```shell
git add .
git commit -m "test Jenkinsfile"
git push origin main
```
![](.2cicd_menggunakan_jenkins/87e57290.png)

setelah itu Jenkins akan mentrigger build secara otomatis
![](.2cicd_menggunakan_jenkins/6ab6d9be.png)

notifikasi berjalan dengan baik
![](.2cicd_menggunakan_jenkins/db85cc8d.png)

# lakukan hal yang sama dengan backend