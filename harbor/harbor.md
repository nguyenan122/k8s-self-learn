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
```
sed -i -e 's/storageClass: ""/storageClass: "nfs-retain"/g' values.yaml
sed -i -e 's/core.harbor.domain/harbor.xxx.vn/g' values.yaml
sed -i 's/className: ""/className: "nginx"/g' values.yaml
Edit file value.yaml and change
-  certSource: "auto" -> certSource: "secret"
-  secretName: "wildcard-tuantls"
```
Result after run sed:
```console
expose:
  type: ingress
...
    secret:
      secretName: "tls.xxx.vn*"
  ingress:
    hosts:
      core: harbor.xxx.vn
...
externalURL: https://harbor.xxx.vn
...
existingSecretAdminPasswordKey: HARBOR_ADMIN_PASSWORD
harborAdminPassword: "Harbor12345"


```
Sửa thêm persistentVolumeClaim là đc:
```console
persistence:
  enabled: true
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:
      existingClaim: ""
      storageClass: "nfs-retain"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 5Gi
      annotations: {}
    jobservice:
      jobLog:
        existingClaim: ""
        storageClass: "nfs-retain"
        subPath: ""
        accessMode: ReadWriteOnce
        size: 1Gi
        annotations: {}
    database:
      existingClaim: ""
      storageClass: "nfs-retain"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 1Gi
      annotations: {}
    redis:
      existingClaim: ""
      storageClass: "nfs-retain"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 1Gi
      annotations: {}
    trivy:
      existingClaim: ""
      storageClass: "nfs-retain"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 5Gi
      annotations: {}
```


```console


helm -n harbor install harbor . --create-namespace
```
### Step 3: How to push image to Harbor
1. Create User
2. Create Project "test"
3. Add User to Project/Member
4. docker login harbor.xxx.vn
5. docker pull nginx:alpine
6. docker tag nginx:alpine harbor.xxx.vn/test/nginx:alpine_v1
7. docker push harbor.xxx.vn/test/nginx:alpine_v1