> #Bài toán: 
> 
> - fresher1 mới vào cty, nằm trong nhóm "developer-readonly" có quyền read-only toàn bộ resource trong namespace "testenv"
> 
> - senior1 cũng thuộc nhóm "developer-readonly". Nhưng có thêm quyền tạo, xóa pod



### 1. Generate key & csr
```console
openssl genrsa -out "fresher1-key.pem" 2048

openssl genrsa -out "senior1-key.pem" 2048

openssl req -new -key "fresher1-key.pem" -out "fresher1-csr.csr" -subj "/CN=fresher1/O=developer"

openssl req -new -key "senior1-key.pem" -out "senior1-csr.csr" -subj "/CN=senior1/O=developer"
```


### 2. Import csr to k8s
```console
cat <<EOF > fresher1-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: fresher1
spec:
  request: $(cat fresher1-csr.csr | base64 -w0)
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

```console
cat <<EOF > senior1-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: senior1
spec:
  request: $(cat senior1-csr.csr | base64 -w0)
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

```console
kubectl apply -f fresher1-csr.yaml

kubectl apply -f senior1-csr.yaml
```


### 3. Load Crt from csr has been approved
```console
kubectl certificate approve fresher1

kubectl certificate approve senior1

kubectl get csr fresher1 -o jsonpath='{.status.certificate}'| base64 -d > fresher1-crt.crt

kubectl get csr senior1 -o jsonpath='{.status.certificate}'| base64 -d > senior1-crt.crt
```


### 4. Grant RBAC "develop group"
```console
kubectl create ns testenv

kubectl -n testenv create role developer-readonly --verb=get,list,watch --resource=*

kubectl -n testenv create rolebinding developer-readonly --role=developer-readonly --group=developer

```

### Grant RBAC addtion for seninor1
```console
kubectl -n testenv create role developer-modify --verb=get,list,watch,delete,create,update,patch --resource=*

kubectl -n testenv create rolebinding developer-modify --role=developer-modify     --user=senior1

```


### 5. Create kube-config
```console
kubectl config set-credentials fresher1 --client-key=fresher1-key.pem --client-certificate=fresher1-crt.crt --embed-certs=true

kubectl config set-credentials senior1 --client-key=senior1-key.pem --client-certificate=senior1-crt.crt --embed-certs=true

kubectl config set-context fresher1 --cluster=kubernetes --user=fresher1

kubectl config set-context senior1 --cluster=kubernetes --user=senior1
```


### 6. Test create fail
```console
k -n testenv --context=fresher1 run nginx --image=nginx

k -n testenv --context=fresher1 expose pod nginx --target-port=80 --port=80 --type=ClusterIP
```
### Test create, list pass
```console
k -n testenv --context=senior1 run nginx --image=nginx

k -n testenv --context=senior1 expose pod nginx --target-port=80 --port=80 --type=ClusterIP

k -n testenv --context=fresher1 get svc

k -n testenv --context=senior1 get svc
```
