## 1. Install

https://fabianlee.org/2022/01/12/kubernetes-nfs-mount-using-dynamic-volume-and-storage-class/?msclkid=a5ca54e2ae7111eca259ced5d2223a6f

https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner 

### Step 1: Install Nfs-server on share server:  
```console
#Install NFS server for CentOS/Redhat
yum install nfs-utils nfs-utils-lib -y
chkconfig rpcbind on
chkconfig nfs on 
service rpcbind restart
service nfs restart

#Install NFS server for Ubuntu
apt-get install nfs-kernel-server -y
systemctl enable nfs-kernel-server
systemctl restart nfs-kernel-server

#Setting NFS
mkdir -p /data/nfs-k8s/delete /data/nfs-k8s/retain
cat << EOF > /etc/exports
/data/nfs-k8s/delete 192.168.88.0/24(rw,sync,subtree_check,no_root_squash)
/data/nfs-k8s/retain 192.168.88.0/24(rw,sync,subtree_check,no_root_squash)
EOF
exportfs -a
showmount -e 127.0.0.1
```

### B2: Install nfs-client for Worker-Node
```console
#Install NFS client for CentOS/Redhat
yum install nfs-utils nfs-utils-lib -y
chkconfig nfs off
chkconfig rpcbind off

#Install NFS client for Ubuntu
apt-get install nfs-common -y
```
### B3: Helm install
```console
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
helm repo update
helm pull nfs-subdir-external-provisioner/nfs-subdir-external-provisioner

tar -xvzf nfs-subdir-external-provisioner-x.x.x.x.tgz
cd nfs-subdir-external-provisioner/

helm install nfs-delete . --set nfs.server=192.168.88.12 \
  --set nfs.path=/data/nfs-k8s/delete \
  --set storageClass.name=nfs-delete \
  --set storageClass.onDelete=Delete \
  --set storageClass.accessModes=ReadWriteMany \
  --create-namespace --namespace nfs-delete

helm install nfs-retain . --set nfs.server=192.168.88.12 \
  --set nfs.path=/data/nfs-k8s/retain \
  --set storageClass.name=nfs-retain \
  --set storageClass.onDelete=Retain \
  --set storageClass.accessModes=ReadWriteMany \
  --create-namespace --namespace nfs-retain
```

### B4: Test
`vim pvc-test.yaml`
```console
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-test
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-delete
  resources:
    requests:
      storage: 2Gi
```
`vim pod-test.yaml`
```console
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  volumes:
  - name: myvol
    persistentVolumeClaim:
      claimName: pvc-test
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh"]
    args: ["-c", "sleep 600000"]
    volumeMounts:
    - name: myvol
      mountPath: /data
```

