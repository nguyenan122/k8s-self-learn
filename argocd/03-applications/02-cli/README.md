## Create Application by cli
```console
argocd login argocd.xxx.vn
k create ns 02-cli

argocd app create -h
argocd app create 02-cli \
--repo git@github.com:nguyenan122/k8s-self-learn.git \
--path argocd/03-applications/02-cli \
--dest-namespace 02-cli \
--dest-server https://kubernetes.default.svc

argocd app sync 02-cli --prune
argocd app list
```