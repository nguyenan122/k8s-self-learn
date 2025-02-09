## 1. Install

https://fabianlee.org/2022/01/12/kubernetes-nfs-mount-using-dynamic-volume-and-storage-class/?msclkid=a5ca54e2ae7111eca259ced5d2223a6f

https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner 

### B1: Cài Nfs server:  
```
yum install nfs-utils nfs-utils-lib -y
chkconfig rpcbind on
chkconfig nfs on 
service rpcbind restart
service nfs restart
mkdir -p /data/nfs-k8s/ ; vim /etc/exports
/data/nfs-k8s/ 192.168.88.0/24(rw,sync,subtree_check,no_root_squash)
exportfs -a
showmount -e 127.0.0.1
```
#Cài cho ubuntu
apt-get install nfs-kernel-server
systemctl enable nfs-kernel-server
systemctl restart nfs-kernel-server


### B2: Cài nfs client trên mỗi worker node, nếu không sẽ lỗi không mount đc vào pod.
```
yum install nfs-utils nfs-utils-lib -y
chkconfig nfs off
chkconfig rpcbind off

#Cài client trên ubuntu
apt-get install nfs-common
```
### B3: Helm install
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
helm pull nfs-subdir-external-provisioner/nfs-subdir-external-provisioner
tar -xvzf nfs-subdir-external-provisioner-x.x.x.x.tgz
cd nfs-subdir-external-provisioner/
k create ns nfs
helm install nfs-provisioner . --set nfs.server=192.168.88.88 \
  --set nfs.path=/data/nfs-k8s/ \
  --set storageClass.name=nfs-provision \
  --set storageClass.onDelete=Delete \
  --set storageClass.accessModes=ReadWriteMany \
  --create-namespace --namespace nfs
```

### B4: Test
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sc-nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-client
  resources:
    requests:
      storage: 2Gi
```

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  volumes:
  - name: myvol
    persistentVolumeClaim:
      claimName: sc-nfs-pvc
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh"]
    args: ["-c", "sleep 600000"]
    volumeMounts:
    - name: myvol
      mountPath: /data
```