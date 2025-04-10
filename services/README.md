
## Mô Tả
Có 4 loại svc:
- ClusterIP: dùng để các pod gọi nhau qua \$(svc_name).$(name_space).svc.cluster.local
- NodePort: listen port trên mỗi worker node.
- LoadBalancer: tạo IP mới loadbalancer
- ExternalName: Ánh xạ tới DNS ngoài giống CNAME hoặc 1 svc khác namespace

### 1.Cluster IP / NodePort
#### 1.1 Compare --port and --target-port

![Images](images/cluster-ip.png)

#### 1.2 Endpoint Static IP outside
Chỉ cần tạo svc và endpoind giống tên là tự động ăn khớp

`vim svc-endpoint.yaml`
```console
apiVersion: v1
kind: Service
metadata:
  name: external-webserver-cka03-svcn
  namespace: kube-public
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 9999
  type: ClusterIP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-webserver-cka03-svcn
  namespace: kube-public
subsets:
- addresses:
  - ip: 192.168.88.200
  ports:
  - port: 9999
    protocol: TCP
```
`kubectl apply -f svc-endpoint.yaml`

### 2.LoadBalancer
Tham khảo: [Metal-LB](https://github.com/nguyenan122/k8s-self-learn/tree/develop/MetalLB)

### 3.ExternalName
Ví dụ về gọi inside cluster, khác namespace
```console
apiVersion: v1
kind: Service
metadata:
  name: external-inside
spec:
  type: ExternalName
  externalName: nginx-blue-svc.nginx-blue.svc.cluster.local
```

Ví dụ về gọi outside:
```console
apiVersion: v1
kind: Service
metadata:
  name: external-outside
spec:
  type: ExternalName
  externalName: test.com.vn
```
ta có thể gọi external-outside với bất cứ port nào -> sẽ được ánh xạ về test.com.vn

