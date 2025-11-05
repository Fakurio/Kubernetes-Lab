# Finalny manifest

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ns-dev
spec: {}
status: {}

---
apiVersion: v1
kind: Namespace
metadata:
  name: ns-prod
spec: {}
status: {}

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ns-dev-quota
  namespace: ns-dev
spec:
  hard:
    cpu: "1"
    memory: 1G
    pods: "10"
status: {}

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ns-prod-quota
  namespace: ns-prod
spec:
  hard:
    cpu: "2"
    memory: 2G
status: {}

---
apiVersion: v1
kind: LimitRange
metadata:
  name: ns-dev-limit-range
  namespace: ns-dev
spec:
  limits:
  - default:
      cpu: 200m
      memory: 256Mi
    defaultRequest:
      cpu: 200m
      memory: 256Mi   
    type: Container
```

# Testy

## 1. Deploy no-test

Po uruchomieniu deploy w eventach nie błędu o przekroczeniu limitu ale po upłynięciu 5m  
komenda ```kubectl get deploy -n ns-dev``` nadal pokazuje w sekcji READY 0/1
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: no-test
  name: no-test
  namespace: ns-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: no-test
  strategy: {}
  template:
    metadata:
      labels:
        app: no-test
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          requests:
            memory: "2Gi"
            cpu: "2000m"
status: {}
```

## 2. Deploy yes-test


Po uruchomieniu deploy komenda ```kubectl get deploy yes-test -n ns-dev``` pokazuje w sekcji READY 1/1,  
a komenda ```kubectl get pod -n ns-dev``` pokazuje poda z przedrostkiem ```yes-test-...``` w statusie RUNNING

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: yes-test
  name: yes-test
  namespace: ns-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: yes-test
  strategy: {}
  template:
    metadata:
      labels:
        app: yes-test
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: 
          requests:
            memory: "128Mi"
            cpu: "100m"
status: {}
```

## 3. Deploy zero-test

Po uruchomieniu deploy komenda ```kubectl get deploy zero-test -n ns-dev``` pokazuje w sekcji READY 1/1,  
a komenda ```kubectl get pod -n ns-dev | grep '^zero-test' | head -n 1 | awk '{print $1}' | xargs -I {} kubectl get pod {} -n ns-dev -o yaml | grep -A5 resources``` pokazuje, że  
pod otrzymał domyślne zasoby narzucone przez limit range  
```
 resources:
      limits:
        cpu: 200m
        memory: 256Mi
      requests:
        cpu: 200m
--
    resources:
      limits:
        cpu: 200m
        memory: 256Mi
      requests:
        cpu: 200m
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: zero-test
  name: zero-test
  namespace: ns-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: zero-test
  strategy: {}
  template:
    metadata:
      labels:
        app: zero-test
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```
