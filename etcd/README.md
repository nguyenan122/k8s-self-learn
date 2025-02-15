

### 1. Command Common 
```
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put
```


### 2. List of key ETCD
```console
ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only
```
### 3. How to get data from worker node
```console
kubectl -n kube-system exec -it etcd-master01 -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only --limit=10"
```
### 4. Check Endpoint Health ETCD
```console
ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key endpoint health

# Result: 127.0.0.1:2379 is healthy: successfully committed proposal: took = 12.914628ms
```

### 5. Backup & Restore  
Install etcd-client
```console
apt-get install etcd-client -y
```
Backup
```console
mkdir -p /opt/backup/etcd/$(date +"%Y%m%d")
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/backup/etcd/$(date +"%Y%m%d")/snapshot.db


# Result: Snapshot saved at /opt/backup/etcd/20250215/snapshot.db
```
Restore
```console
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key \
  --data-dir <data-dir-location-restore> snapshot restore /opt/backup/etcd/20250215/snapshot.db
```
