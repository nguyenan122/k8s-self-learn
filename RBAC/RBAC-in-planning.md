
### 5. Chia quyền trên production và Development

Dev ở trên prod chỉ xem đc ["pods", "deployments", "services", "configmaps"] và pod/logs. Còn ở trên môi trường dev thì phá thỏa mái
```console
# 1. Create a restricted developer role for production
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prod-developer
  namespace: production
rules:
# Allow read-only access to most resources
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
# Allow logs access for debugging
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
```
```console
# 2. Create a more permissive role for development
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-developer
  namespace: development
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "services", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```


### 1. Quyền reader pod/node
```console
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]  # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

```console
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list"]
```

### 2. Quyền devops team
```console
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: devops-role
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "create", "update"]
```

### 3. Quyền develop-team và lead-team
```console
# 2. Create a Role for team developers
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: team-workspace
  name: team-developer
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get", "list", "watch"]
```

```console
# 3. Create a Role for team leads(wider permissions)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: team-workspace
  name: team-lead
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["*"]
```

### 4. Quyền Infra-manager

```console
# 1. Create a ClusterRole for platform engineers
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform-engineer
rules:
# Allow management of core cluster resources
- apiGroups: [""]
  resources: ["nodes", "namespaces", "persistentvolumes"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
# Allow management of storage classes
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
# Allow management of network policies
- apiGroups: ["networking.k8s.io"]
  resources: ["networkpolicies"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
```

Nguồn: https://itnext.io/mastering-rbac-in-kubernetes-a-complete-guide-with-practical-examples-1bc24991e7c1 