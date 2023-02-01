# setup cloudflare untuk backend
pergi ke situs cloudflare > pilih akun > website > klik
domain yang tersedia
![](.cloud_computing_images/36ecf02e.png)

pilih dns > add record > masukan domain dan ip public dari gateway
![](.2setup_backend_and_database_images/f83c9546.png)

# webserver (setup domain untuk backend)

pada server gateway, buat file konfigurasi baru di direktori 
/etc/nginx/sites-enabled

isi file konfigurasi kurang lebih berikut
```shell
server {
    server_name api.reiya.my.id;

    location / {
             proxy_pass http://10.116.106.209:5000;
    }
}
```
![](.2setup_backend_and_database_images/25269dc9.png)
sesuaikan domain dan ip private dari appserver

cek syntax nginx
```shell
sudo nginx -t
```
![](.2setup_backend_and_database_images/9f0ed0a5.png)

restart / reload nginx untuk memperbarui perubahan
```shell
sudo systemctl restart nginx
```
![](.2setup_backend_and_database_images/76a7546a.png)

# setup domain frontend untuk menghubungkan ke backend
pada appserver, di folder wayshub_frontend/config/api.js, ubah baseURL
ke domain backend kita yang sudah dibuat
```shell
import axios from 'axios';

const API = axios.create({
    baseURL: "http://api.reiya.my.id/api/v1"
});

const setAuthToken = (token) => {
    if(token){
        API.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    } else {
        delete API.defaults.headers.common['Authorization'];
    }
}

export {
    API,
    setAuthToken
}
```
![](.2setup_backend_and_database_images/9ac65a56.png)

# setup mysql database
pada appserver, install mysql dengan menggunakan perintah
```shell
sudo apt install mysql-server -y
```
![](.2setup_backend_and_database_images/93cf7f7d.png)

kita tidak dapat langsung menjalankan mysql_secure_instalation karena 
dapat menyebabkan recrusive looping saat form pengisian password root, 
untuk mencegahnya, kita perlu seting password secara manual.

jalankan mysql dengan sudo
```shell
sudo mysql
```
![](.2setup_backend_and_database_images/1d2556dd.png)

ubah password root dengan menggunakan perintah dibawah, sesuaikan password 
dengan kebutuhan
```shell
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'kata_sandi';
```
![](.2setup_backend_and_database_images/8e8e0847.png)

untuk menyimpan perubahan, gunakan perintah
```shell
FLUSH PRIVILEGES;
```
![](.2setup_backend_and_database_images/65a932b6.png)

exit mysql mengunakan perintah exit; lalu kita baru bisa menjalankan mysql_secure_instalation 
tanpa memiliki maslah
```shell
sudo mysql_secure_installation
```

masukan password root yang sebelumnya sudah disetel image

![](.2setup_backend_and_database_images/30e9ae49.png)

setelah itu, pilih y untuk mengatur setingan password validation 

![](.2setup_backend_and_database_images/02851934.png)

pilih 0 untuk mengatur password validationnya menjadi low 
(agar password bisa diisi tanpa kombinasi nomor, karakter spesial, dan mixed case)

![](.2setup_backend_and_database_images/b909aa8f.png)

input y untuk mengubah root password, atau sembarang kata untuk tidak
![](.2setup_backend_and_database_images/f2f2ee7a.png)

pilih y untuk menghapus anonymous user (anonymous user memungkinkan 
kita untuk masuk ke mysql tanpa mempunyai akun)

![](.2setup_backend_and_database_images/2540cafa.png)

pilih y untuk mematikan login secara remote 

![](.2setup_backend_and_database_images/3cc50a02.png)

pilih y untuk menghapus database tes 

![](.2setup_backend_and_database_images/a655d62a.png)

pilih y untuk mereload setingan mysql

![](.2setup_backend_and_database_images/a2dba9b2.png)

setelah selesai, kita masuk ke mysql prompt menggunakan 
user root, jalankan
```shell
sudo mysql -u root -p
```
![](.2setup_backend_and_database_images/9b7e0e6b.png)

buat user baru menggunakan perintah
```shell
CREATE USER 'nama_user_baru'@'%' IDENTIFIED BY 'isi_password';
```
![](.2setup_backend_and_database_images/728c5495.png)

beri perizinan penuh kepada user baru kita
```shell
GRANT ALL PRIVILEGES ON * . * TO 'reiya''@''%';
```

![](.2setup_backend_and_database_images/eac6f5d8.png)

flush privileges untuk menyimpan semua perubahan

![](.2setup_backend_and_database_images/c12705ec.png)

# setup wayshub backend
clone wayshub backend
```shell
git clone https://github.com/dumbwaysdev/wayshub-backend
```
![](.2setup_backend_and_database_images/2dde7665.png)

setup konfigurasi backend dengan database di direktori config/config.json

```shell
{
  "development": {
    "username": "reiya",
    "password": "kata_sandi",
    "database": "dbwayshub",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```
![](.2setup_backend_and_database_images/d811aa5a.png)

install sequelize-cli
```shell
npm install -g sequelize-cli
```
![](.2setup_backend_and_database_images/1d6bad4c.png)

buat database dbwayshub menggunakan 
```shell
sequelize db:create
```
![](.2setup_backend_and_database_images/f7d1e9e1.png)

generate isi dari database
```shell
sequelize db:migrate
```
![](.2setup_backend_and_database_images/7f5f4773.png)

generate script pm2
```shell
pm2 ecosystem simple
```
![](.2setup_backend_and_database_images/bb2d3a44.png)

buka file ecosystemnya, nama disesuaikan keinginan

```shell
module.exports = {
  apps : [{
    name   : "wayshub_backend",
    script : "npm run start"
  }]
}
```
![](.2setup_backend_and_database_images/35705184.png)

jalankan pm2
```shell
pm2 start
```
![](.2setup_backend_and_database_images/9c3fdfae.png)

integrasi frontend dengan backend berhasil

![](.2setup_backend_and_database_images/24eff5c4.png)

![](.2setup_backend_and_database_images/d63c0f3e.png)

![](.2setup_backend_and_database_images/0b700c31.png)

# pemasangan https

pada server gateway update snap untuk mendapatkan package terbaru
```shell
sudo snap install core; sudo snap refresh core
```
![](.2setup_backend_and_database_images/c95fe3fc.png)

install certbot
```shell
sudo snap install --classic certbot
```
![](.2setup_backend_and_database_images/b6a35255.png)

tambahkan path certbot agar perintah certbot dapat dijalankan
```shell
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
![](.2setup_backend_and_database_images/552a40f6.png)

seting agar certbot dapat menggunakan plugin
```shell
sudo snap set certbot trust-plugin-with-root=ok
```
![](.2setup_backend_and_database_images/2d57fc28.png)

install certbot dns plugin
```shell
sudo snap install certbot-dns-nama_dns_provider
```
![](.2setup_backend_and_database_images/5c3100f9.png)

buat file untuk menyimpan API token
```shell
nano /home/reiya24/.my_config/cloudflare.ini
```
isi file
```shell
dns_cloudflare_email = "nama_email@nama_email.com"
dns_cloudflare_api_key = "nomor_api_token"
```
![](.2setup_backend_and_database_images/4add3142.png)

untuk mengambil global token, pergi ke dashboard cloudflare > my
profile > API tokens> view global API keys 
![](.2setup_backend_and_database_images/f42251e8.png)

jalankan certbot
```shell
certbot -i nginx \
  --dns-cloudflare \
  --dns-cloudflare-credentials lokasi_API-key \
  -d domain_yang_ingin_didaftarkan \
  -d *.domain_yang_ingin_didaftarkan
```
pilih y

![](.2setup_backend_and_database_images/c46ecbb0.png)

pemasangan SSL berhasil

# firewall
pada kedua server kita, nyalakan firewall
```shell
sudo ufw enable
```
![](.2setup_backend_and_database_images/13bcd044.png)

![](.2setup_backend_and_database_images/3ecca16d.png)

buka port-port yang dibutuhkan
```shell
sudo ufw allow nomor_port
```
![](.2setup_backend_and_database_images/67e79143.png)
![](.2setup_backend_and_database_images/307a968f.png)