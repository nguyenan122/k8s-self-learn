
Refer: https://coredns.io/plugins/hosts/

## "Specific IP" resolve CoreDNS
### 1. Create pod test lookup 
```console
k run busybox --image=busybox -- sleep 36000
k exec -it busybox -- nslookup dantri.com.vn

> Result: 
Name:   dantri.com.vn
Address: 183.81.34.144
Name:   dantri.com.vn
Address: 183.81.34.143
```

### 2. Change config file CoreDNS

`k -n kube-system edit configmaps coredns`

```console
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        hosts example.hosts dantri.com.vn vnexpress.net {
           192.168.88.90 dantri.com.vn
           192.168.88.90 vnexpress.net
           fallthrough
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }

```
Explain: 
Load example.hosts file and only serve "dantri.com.vn" and "vnexpress.net" from it and fall through to the next plugin if query doesn’t match.

### 3. Test again:
```console
k exec -it busybox -- nslookup dantri.com.vn
Server:         10.96.0.10
Address:        10.96.0.10:53


Name:   dantri.com.vn
Address: 192.168.88.90

```