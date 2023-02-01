# membuat virtual private server di IDCloudHost

masuk ke dashborad IDcloudhost https://console.idcloudhost.com

buat virtual 2 virtual private server untuk appserver dan gateway
![](.cloud_computing_images/f773fa4c.png)

generate SSH key
```shell
ssh-keygen
```
![](.cloud_computing_images/5e4ba672.png)

output pertama akan meminta lokasi dimana file ssh akan disimpan,
input lokasi yang diinginkan. atau enter untuk membiarkan default

output kedua akan meminta passphrase, saya memilih untuk mengkosongkannya
dengan menekan tombol enter

otomatis akan ada 2 file baru di direktori tersebut, 1 sebagai kunci ssh, 
1 lagi sebagai gembok ssh (berekstensikan .pub)

Setelah file SSH digenerate, kita gunakan ssh-copy-id untuk menaruh file
public key kita ke dalam virtual machine

![](.cloud_computing_images/20055c48.png)

# menjalankan aplikasi wayshub-frontend menggunakan nodeJS

akses server menggunakan SSH

```shell
ssh -i lokasi_file_kunci_ssh username@ip_publik
```
![](.cloud_computing_images/58e7fef2.png)

jalankan system update dan upgrade (opsional)

```shell
sudo apt update && sudo apt upgrade -y
```

![](.cloud_computing_images/6147b237.png)

clone wayshub frontend
```shell
git clone https://github.com/dumbwaysdev/wayshub-frontend
```
![](.cloud_computing_images/c9043c45.png)

copy script node installer dari komputer lokal ke server
```shell
scp -i kunci_ssh nama_file username@ip_public:lokasi_hasil_salinan
```
![](.cloud_computing_images/63d8af8a.png)

isi file node installer
```shell
#!/bin/bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
eval "$(cat ~/.bashrc | tail -n +10)"
nvm install 16
nvm use 16
node -v
npm install pm2 -g
exec bash
```
![](.cloud_computing_images/a346f495.png)

jalankan scriptnya
```shell
./node_installer.sh
```
![](.cloud_computing_images/78e8599d.png)

pada direktori wayshub frontend lakukan npm install untuk menginstall
semua depedensi yang diperlukank
```shell
npm install
```
![](.cloud_computing_images/14ca7a26.png)

generate script pm2 agar mudah dijalankan
```shell
pm2 ecosystem simple
```
![](.cloud_computing_images/83fc391e.png)

buka file ecosystem.config.js lalu ubah konfigurasinya.
nama bebas untuk script ketikan npm run start
```shell
module.exports = {
  apps : [{
    name   : "wayshub frontend",
    script : "npm run start"
  }]
}
```
![](.1cloud_computing_images/3a653ebf.png)

jalankan file konfigurasi pm2
```shell
pm2 start
```
![](.1cloud_computing_images/ebde380e.png)

buka web browser dan ketikan IP publik:3000
![](.cloud_computing_images/bc4ba4c7.png)

# setup domain di cloudflare

pergi ke situs cloudflare > pilih akun > website > klik
domain yang tersedia
![](.cloud_computing_images/36ecf02e.png)

pilih dns > add record > masukan domain dan ip public dari gateway
![](.cloud_computing_images/fa478f1a.png)

# reverse proxy nginx
akses server menggunakan ssh
update & upgrade system (opsional
```shell
sudo apt update && sudo apt upgrade -y
```
![](.cloud_computing_images/9f6c49b1.png)

install nginx
```shell
sudo apt install nginx -y
```
![](.cloud_computing_images/54a0005c.png)

buat file konfigurasi ngix di /etc/nginx/sites-enabled dengan akses root
```shell
sudo nano /etc/nginx/sites-enabled
```

isikan kurang lebih berikut
```shell
server {
    server_name reiya.my.id;

    location / {
             proxy_pass http://10.116.106.209:3000;
    }
}
```
![](.cloud_computing_images/3974cb98.png)
sesuaikan server_name dengan domain yang ada, dan
proxy pass masukan ip private dari appserver

gunakan perintah dibawah untuk mengecek apakah ada kesalahan syntax
di file konfigurasi ngix
```shell
sudo nginx -t
```
![](.cloud_computing_images/bcde7073.png)

restart nginx untuk memperbarui perubahan konfigurasi
```shell
sudo systemctl restart nginx
```
![](.cloud_computing_images/4ed133f3.png)

akses domain
![](.cloud_computing_images/15ba0a3b.png)

# menyalakan firewall

## webserver
nyalakan firewall menggunakan perintah
```shell
sudo ufw enbale
```
![](.cloud_computing_images/a4595bf4.png)

tambahkan rule untuk memberi akses ke port 22 untuk ssh, dan 80 untuk nginx
![](.cloud_computing_images/75562144.png)

## appserver
gunakan perintah yang sama, namun port yang dibuka adalah 22 dan 3000
![](.cloud_computing_images/a9926f5d.png)

