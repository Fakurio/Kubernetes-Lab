# Sprawozdanie

1. Utworzenie przestrzeni nazw remote
```yaml
kubectl create ns remote
```

2. Utworzenie poda w przestrzeni remote oraz pliku yaml dla serwisu który go udostępni
```yaml
kubectl run remoteweb --image=nginx -n=remote
```
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    # odwołanie do poda remoteweb
    run: remoteweb
  name: remoteweb
  namespace: remote
spec:
  ports:
  - name: 80-80
    nodePort: 31999
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    # odwołanie do poda remoteweb
    run: remoteweb
  type: NodePort
status:
  loadBalancer: {}
```

3. Utworzenie poda w przestrzeni domyślnej
```yaml
kubectl run testpod --image=busybox -- sleep infinity
```

4. Sprawdzenie łączności między podami

Za pomocą polecenia `kubectl get svc remoteweb -n remote` sprawdzilem ip klastra serwisu remoteweb i następnie sprawdziłem połączenie komendą  
`kubectl exec testpod -it -- wget 10.100.239.85:80 --spider --timeout=1`.  
W rezultacie dostałem taką odpowiedź  
`Connecting to 10.100.239.85:80 (10.100.239.85:80)
remote file exists` co potwierdza, że pody mogą się komunikować.

5. Sprawdzenie łączności z podem remoteweb z zewnątrz

Za pomocą polecenia `minikube service remoteweb -n remote --url &` dostałem adres `http://127.0.0.1:40207` który użyłem w komendzie `wget` do pobrania głównej strony z nginxa uruchomionego w podzie remoteweb.  
Operacja powiodła się co znaczy, że nodePort w serwisie remoteweb działa poprawnie.
