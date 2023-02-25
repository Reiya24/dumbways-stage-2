# tambahkan public ke ke akun gitlab
pada [halaman gitlab](https://gitlab.com/) 
klik foto profile > preferences > SSH keys, tambahkan public key

# membuat repository di gitlab

pada [halaman gitlab](https://gitlab.com/), pilih new project > 
create blank project > masukan deskripsi project
![](.3cicd_menggunakan_gitlab_ci_images/3c14894a.png)


# setup git di appserver
atur name dan email di git config
```shell
git config --global user.name "nama"
```
```shell
git config --global user.email "password"
```
![](.3cicd_menggunakan_gitlab_ci_images/f135b1a1.png)

inisalisasi direktori agar dikenali git
```shell
git init
```
tambahkan remote branch gitlab
```shell
git remote add origin git@gitlab.com:Reiya24/literature-backend.git
```
![](.3cicd_menggunakan_gitlab_ci_images/575e7f1d.png)

tambahkan dan pindah branch main
```shell
git checkout -b main
```
![](.3cicd_menggunakan_gitlab_ci_images/e06b3794.png)

tambahkan semua file ke staging area
```shell
git add .
```
![](.3cicd_menggunakan_gitlab_ci_images/e8aaa6c6.png)

lakukan commit
```shell
git commit -m "Initial commit"
```
![](.3cicd_menggunakan_gitlab_ci_images/3ba2e24f.png)

push ke gitlab repository
```shell
git push origin main
```
![](.3cicd_menggunakan_gitlab_ci_images/a7fbeff4.png)

# membuat pipeline
pada direktori frontend di appserver, buat file pipeline yang bernama
.gitlab-ci.yml

![](.3cicd_menggunakan_gitlab_ci_images/613c3cf5.png)

# tambahkan variable gitlab CI
pada repository gitlab, pilih setting > CI/CD
![](.3cicd_menggunakan_gitlab_ci_images/baab093f.png)

pilih variable
![](.3cicd_menggunakan_gitlab_ci_images/eb463327.png)

tambahkan variable, variable yang dibutuhkan pada kasus ini adalah
$USERNAME, $PASSWORD_DOCKER, $IMAGE_NAME, $PRIVATE_KEY, $VM_ADDRESS
![](.3cicd_menggunakan_gitlab_ci_images/93843249.png)

push pipeline ke repository github
![](.3cicd_menggunakan_gitlab_ci_images/0caeea69.png)

pipeline secara otomatis akan berjalan
![](.3cicd_menggunakan_gitlab_ci_images/48a3580f.png)

pipeline berhasil dijalankan
