
Source: https://kubernetes.github.io/ingress-nginx/deploy/

---
### Install from helm

```console
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx 
helm repo update
helm pull ingress-nginx/ingress-nginx
tar -xvzf ingress-nginx-4.12.0.tgz
cd ingress-nginx/
k create ns ingress-nginx
helm install ingress-nginx . --namespace ingress-nginx --create-namespace
```
Test:
```console
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
kubectl create ingress demo-localhost --class=nginx --rule=demo.localdev.me/*=demo:80

# curl --resolve demo.localdev.me:30188:127.0.0.1 http://demo.localdev.me:30188
<html><body><h1>It works!</h1></body></html>
```
---
### Install without helm
If you don't have Helm or if you prefer to use a YAML manifest, you can run the following command instead:

```console
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
```
Test:
```console
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
kubectl create ingress demo-localhost --class=nginx --rule=demo.localdev.me/*=demo:80

# curl --resolve demo.localdev.me:30188:127.0.0.1 http://demo.localdev.me:30188
<html><body><h1>It works!</h1></body></html>
```