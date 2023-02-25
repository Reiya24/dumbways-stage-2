# frontend
buat file konfigurasi
```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  selector:
    matchLabels:
      app: frontend-label
  replicas: 1
  template:
    metadata:
      labels:
        app: frontend-label
    spec:
      containers:
        - name: frontend-container
          image: reiya24/literature-frontend:latest
          stdin: true
          stdinOnce: false
          tty: true
          ports:
            - containerPort: 3000

---

apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend-label
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30000
```
![](.2deploy_frontend_dan_backend_images/35ff89ed.png)

jalankan file konfigurasi
```shell
kubectl apply -f nama_file.yaml
```
![](.2deploy_frontend_dan_backend_images/37ae9cef.png)

expose service nodeport ke host
```shell
minikube service nama_service
```
![](.2deploy_frontend_dan_backend_images/513404fa.png)


![](.2deploy_frontend_dan_backend_images/a2d9ef6e.png)

# backend

```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  selector:
    matchLabels:
      app: backend-label
  replicas: 1
  template:
    metadata:
      labels:
        app: backend-label
    spec:
      containers:
        - name: backend-container
          image: naninanides/literature-backend:latest
          stdin: true
          stdinOnce: false
          tty: true
          ports:
            - containerPort: 5000

---

apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: NodePort
  selector:
    app: backend-label
  ports:
    - port: 5000
      targetPort: 5000
      nodePort: 32500
```
![](.2deploy_frontend_dan_backend_images/924a034d.png)

jalankan file konfigurasi
```shell
kubectl apply -f nama_file.yaml
```
![](.2deploy_frontend_dan_backend_images/64a64a84.png)

expose service
```shell
minikube service nama_service
```
![](.2deploy_frontend_dan_backend_images/77b108a2.png)

![](.2deploy_frontend_dan_backend_images/eaaeab75.png)