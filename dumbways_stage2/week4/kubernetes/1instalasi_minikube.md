# pada sistem operasi berbasis arch linux

install depedensi yang diperlukan
```shell
sudo pacman -Syu libvirt qemu ebtables dnsmasq
```
![](.1instalasi_minikube_images/49d6a169.png)

jalankan service yang diperlukan
```shell
sudo systemctl enable  libvirtd.service --now && sudo systemctl enable virtlogd.service --now
```
![](.1instalasi_minikube_images/f9f6adc7.png)

instal kvm2-driver, minikube dan kubectl
```shell
yay -S docker-machine-driver-kvm2 minikube kubectl-bin
```
![](.1instalasi_minikube_images/63cfbfc4.png)

restart komputer

jalankan minikube
```shell
minikube start
```
![](.1instalasi_minikube_images/1c27cdd4.png)

