# Sprawozdanie

Chart mariadb od CloudPirates domyślnie uruchamia bazę danych w klastrze jako StatefulSet,  
dlatego w pliku z customowymi wartościami wystarczy tylko ustawić service na headless  
```yaml
service:
  clusterIP: "None"
``` 

Po zainstalowaniu charta komendą  
`helm install my-mariadb oci://registry-1.docker.io/cloudpirates/mariadb -f mariadb-values.yaml`  

Pod z bazą, service oraz StatefulSet zostały poprawnie uruchomione  

![title](images/verification.png) 
