# Sprawozdanie
W pierwszej kolejności wygenerowalem manifest komendą `kubectl create deploy nginx-proxy --image=nginx:1.19.1 -r=5 --dry-run=client -o yaml > solution.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-proxy
  name: nginx-proxy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-proxy
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-proxy
    spec:
      containers:
      - image: nginx:1.19.1
        name: nginx
        resources: {}
status: {}
```
Następnię dodałem etykietę `typ=proxy` wraz z adnotacją
```yaml 
metadata:
  labels:
    app: nginx-proxy
    typ: proxy
  name: nginx-proxy
  annotations:
    kubernetes.io/description: "Proxy na bazie nginx"
```
oraz informacje na temat strategii update'u konfiguracji.
```yaml
strategy: 
    type: RollingUpdate 
    rollingUpdate:
      maxUnavailable: 2 
```

## Finalny manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-proxy
    typ: proxy
  name: nginx-proxy
  annotations:
    kubernetes.io/description: "Proxy na bazie nginx"
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-proxy
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
  template:
    metadata:
      labels:
        app: nginx-proxy
    spec:
      containers:
      - image: nginx:1.19.1
        name: nginx
        resources: {}
status: {}
```
