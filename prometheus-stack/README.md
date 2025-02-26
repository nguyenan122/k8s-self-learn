

## Install from HELM


Add helm repo:  
```console
https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

```

Pull kube-prometheus-stack and extract:
```console
mkdir prometheus-stack
cd prometheus-stack/
helm pull prometheus-community/kube-prometheus-stack
tar -xvzf kube-prometheus-stack-XX.X.X.tgz #2025-01 with version = 69.2.0
cd kube-prometheus-stack/
```

`vim values.yaml
`
- Enable ingress Alertmanager (Line 607)
```console
  ingress:
    enabled: true
    ingressClassName: nginx
    annotations: {}
    labels: {}
    hosts:
      - alertmanager.xxx.vn
    paths: []
    tls:
    - secretName: wildcard-tuantls
      hosts:
      - alertmanager.xxx.vn
```
- Enable ingress Grafana (Line 1260)
```console
  ingress:
    enabled: true
    ingressClassName: nginx
    annotations: {}
    labels: {}
    hosts:
      - grafana.xxx.vn
    path: /
    tls:
    - secretName: wildcard-tuantls
      hosts:
      - grafana.xxx.vn
```
- Enable ingress Prometheus (Line 3619)
```console
  ingress:
    enabled: true
    ingressClassName: nginx
    annotations: {}
    labels: {}
    hosts:
      - prometheus.xxx.vn
    paths: []
    tls:
    - secretName: wildcard-tuantls
      hosts:
      - prometheus.xxx.vn
```


- use nfs-provision for Prometheus (Line 4215)
```console
    storageSpec:
     volumeClaimTemplate:
       spec:
         storageClassName: nfs-retain
         accessModes: ["ReadWriteOne"]
         resources:
           requests:
             storage: 10Gi
```
Sample file: 


`k create ns monitoring`   
`helm -n monitoring install kps .`



### Nginx proxy to stack (test-only)
```console
server {
    listen 80;

    server_name _;

    location / {
        proxy_pass http://192.168.88.188/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 443 ssl;
    server_name _;

    ssl_certificate /etc/nginx/ca.crt;
    ssl_certificate_key /etc/nginx/ca.key;

    location / {
        proxy_pass https://192.168.88.188/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Add more target into Prometheus
find keywork in values.yaml file: `additionalScrapeConfigs`