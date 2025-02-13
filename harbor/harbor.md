Source: https://github.com/goharbor/harbor-helm  
https://vishynit.medium.com/setting-up-harbor-registry-on-kubernetes-using-helm-chart-5989d7c8df2a

### Step 1: Add the Harbor Helm Repository

```console
helm repo add harbor https://helm.goharbor.io
helm repo update
kubectl create namespace harbor
helm pull  harbor/harbor
tar -xvzf harbor-x.xx.x.tgz  #(2025-02 harbor-1.16.2.tgz)
cd harbor/
```

### Step 2: Modify values.yaml
```console
expose:
  type: ingress
...
    secret:
      secretName: "tuannamevn"
  ingress:
    hosts:
      core: harbor.xxx.vn
...
externalURL: https://harbor.xxx.vn
...
existingSecretAdminPasswordKey: HARBOR_ADMIN_PASSWORD
harborAdminPassword: "Harbor@12345"
```

```console
helm -n harbor install harbor .
```
### Step 3: How to push image to Harbor
1. Create User
2. Create Project "test"
3. Add User to Project/Member
4. docker login harbor.xxx.vn
5. docker pull nginx:alpine
6. docker tag nginx:alpine harbor.xxx.vn/test/nginx:alpine_v1
7. docker push harbor.xxx.vn/test/nginx:alpine_v1