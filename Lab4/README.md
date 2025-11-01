## Pierwsza wersja manifestu

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: loop
  name: loop
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - for i in {1..5}; do echo "A piece of cake!"; done
    image: busybox:1.36.1
    name: loop
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Finalna wersja manifestu

Aby pod utworzony z manifestu był ciągle w stanie Running należy użyć pętli nieskończonej w poleceniu powłoki którą ma wykonywać kontener w podzie.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: loop
  name: loop
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - while true; do echo "A piece of cake!"; done
    image: busybox:1.36.1
    name: loop
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
