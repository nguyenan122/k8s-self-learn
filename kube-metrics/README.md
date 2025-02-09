
Source: https://github.com/kubernetes-sigs/metrics-server/

```console
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml 
```
`vim components.yaml`
```console
 - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-insecure-tls  #ADD_THIS_LINE
        - --kubelet-preferred-address-types=InternalIP
        image: k8s.gcr.io/metrics-server-amd64:v0.3.6
```

`k -n kube-system apply -f components.yaml`

Test:
```console
k top node
k top pod -A
```