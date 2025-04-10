
### 1. Mô tả
Sự khác biệt giữa Canary và Blue-Green:
- Canary sẽ chỉ lái 1 phần người dùng đi vào deployment mới, pod mới (ví dụ 10%). Tránh ảnh hưởng rui ro trên diện rộng
- Blue/Green là 2 production giống hệt nhau. Sau khi môi trường phụ triển khai code mới, sẽ được lái toàn bộ 100% người dùng sang luôn.


### 2. Triển khai deployment

```console
kubectl create namespace nginx-blue
kubectl create namespace nginx-green
```

`# vim blue-app.yaml`
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-blue-svc
  namespace: nginx-blue
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-blue-config
  namespace: nginx-blue
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Blue Deployment</title>
      <style>
      body {
        background-color: blue;
      }
      </style>
    </head>
    <body>
      <h1>Blue Deployment</h1>
    </body>
    </html>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-blue-deployment
  namespace: nginx-blue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config
              mountPath: /usr/share/nginx/html/index.html
              subPath: index.html
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-blue-config
```            

`vim green-app.yaml`
```console
apiVersion: v1
kind: Service
metadata:
  name: nginx-green-svc
  namespace: nginx-green
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-green-config
  namespace: nginx-green
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Green Deployment</title>
      <style>
      body {
        background-color: green;
      }
      </style>
    </head>
    <body>
      <h1>Green Deployment</h1>
    </body>
    </html>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-green-deployment
  namespace: nginx-green
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config
              mountPath: /usr/share/nginx/html/index.html
              subPath: index.html
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-green-config
```

```
kubectl apply -f green-app.yaml -n nginx-green
kubectl apply -f blue-app.yaml -n nginx-blue
```

### 4. Canary
Tạo 2 ingress giống hệt nhau về host. Khác nhau chỉ là 1 cái thêm annotation. Muốn thay đổi tỉ lệ weight thì sửa lại % là xong.

```console
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: canary-green-ingress
  namespace: nginx-green
spec:
  ingressClassName: nginx
  rules:
    - host: "canary.test.com"
      http:
        paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: nginx-green-svc
                port:
                  number: 80
```


```console
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: canary-blue-ingress
  namespace: nginx-blue
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  ingressClassName: nginx
  rules:
    - host: "canary.test.com"
      http:
        paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: nginx-blue-svc
                port:
                  number: 80
```

### 4. Test tải
`vim get.sh`
```
#!/bin/bash
TOTAL=1000  # Number of requests to make
counter=0
blue=0
green=0

while [ $counter -lt $TOTAL ]; do
  response=$(curl -s http://canary.test.com)
  
  if [[ $response == *"Blue Deployment"* ]]; then
    ((blue++))
  elif [[ $response == *"Green Deployment"* ]]; then
    ((green++))
  fi
  
  ((counter++))
  
  # Print progress
  echo -ne "Blue: $blue ($(( blue * 100 / counter ))%), Green: $green ($(( green * 100 / counter ))%), Total: $counter\r"
  
  sleep 0.1
done

echo -e "\nFinal split:"
echo "Blue: $blue ($(( blue * 100 / TOTAL ))%)"
echo "Green: $green ($(( green * 100 / TOTAL ))%)"
```

```console
# Kết quả test:
./get.sh
Blue: 32 (19%), Green: 130 (80%), Total: 162
```



-----
Refer: https:/itnext.io/kubernetes-canary-the-art-of-zero-downtime-deployments-2ab03aa0ffee