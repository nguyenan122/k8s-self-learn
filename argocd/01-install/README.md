### 1.1 Install
Cài đặt Argocd Server https://argo-cd.readthedocs.io/en/stable/getting_started/ 
```console
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# lấy mật khẩu để vào admin
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Apply SSL cho ArgoCD
k -n argocd create secret tls argocd-server-tls --cert=ca.crt --key=ca.key
```

### 1.2 Cài đặt ArgoCD-CLI
```
https://github.com/argoproj/argo-cd/releases 
wget https://github.com/argoproj/argo-cd/releases/download/v2.14.3/argocd-linux-amd64
mv argocd-linux-amd64 argocd
mv argocd /usr/local/bin/
chmod +x /usr/local/bin/argocd

# Argocd Completion for argocd-cli
argocd completion bash > /etc/bash_completion.d/argocd
```

### 1.3 Ingress cho ArgoCD
Tạo ingress và ssl
```console
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.xxx.vn
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
  tls:
  - hosts:
    - argocd.xxx.vn
    secretName: argocd-server-tls
```
