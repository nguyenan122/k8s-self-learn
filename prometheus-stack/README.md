

## Install from HELM


Add helm repo:  
```console
https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

```

Pull kube-prometheus-stack and extract:
```console
mkdir prometheus-stack
cd prometheus-stack/
helm pull prometheus-community/kube-prometheus-stack
tar -xvzf kube-prometheus-stack-XX.X.X.tgz #2025-01 with version = 69.2.0
cd kube-prometheus-stack/
```

`vim values.yaml
`
```

```

