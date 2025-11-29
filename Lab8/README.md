# Sprawozdanie

## Przygotowanie środowiska

Utworzenie nodów  
`minikube start --nodes 4 -p 4-nodes --cni=calico`

Dodanie label do poszczególnych nodów  
`kubectl label nodes 4-nodes-m02 app=frontend`  
`kubectl label nodes 4-nodes-m03 app=backend`  
`kubectl label nodes 4-nodes-m04 app=database`

## Przygotowanie plików yaml

### Frontend

#### frontend-deploy.yaml  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: frontend
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  strategy: {}
  template:
    metadata:
      labels:
        app: frontend
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: app
                operator: In
                values:
                - frontend
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

#### frontend-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  type: NodePort
  selector:
    app: frontend 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### Backend

#### backend-deploy.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: backend
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  strategy: {}
  template:
    metadata:
      labels:
        app: backend
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: app
                operator: In
                values:
                - backend 
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

#### backend-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP
  selector:
    app: backend 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### Database

#### database-deploy.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: database
  name: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  strategy: {}
  template:
    metadata:
      labels:
        app: database
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: app
                operator: In
                values:
                - database
      containers:
      - image: mysql
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root
        resources: {}
status: {}
```

#### database-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database-svc
spec:
  type: ClusterIP
  selector:
    app: database 
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

### Polityka sieciowa

Dodajemy polityke tylko dla podów z label app=database i zezwalamy na ruch wejściowy  
tylko z podów z label app=backend na porcie 3306. Wykorzystujemy tutaj domyślne zachowanie  
K8 który blokuje pozostały ruch ingress i egress kiedy do danego zasobu jest przypisana  
polityka sieciowa.

#### netpol.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: control-access-db
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 3306
```

## Testy

Do testowania wykorzystałem kontenery efemeryczne

#### Frontend

Podpinamy się do jednego z podów frontend  
`kubectl debug -it frontend-868cc6c9b8-68q4t --image=busybox:1.28`  

Testujemy połączenie z serwisem bazy danych  
`nc -zv database-svc 3306`  

W konsoli ciągle miga kursor zachęty co świadczy o blokadzie połączenia i ciągłych próbach  
jego nawiązania

#### Backend

Podpinamy się do poda backend
`kubectl debug -it backend-67c9c6586-4tdmp --image=busybox:1.28`

Testujemy połączenie
`nc -zv database-svc 3306`

W wyniku otrzymujemy, że połączenie zostało nawiązane  
`database-svc (10.106.1.151:3306) open`

Testujemy na innym porcie  
`nc -zv database-svc 4000`

Wynik taki sam jak w przypadku poda frontend

