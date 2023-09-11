# Домашнее задание к занятию «Базовые объекты K8S»

## Задание 1. Создать Pod с именем hello-world

1. Создать манифест (yaml-конфигурацию) Pod.
2. Использовать image - gcr.io/kubernetes-e2e-test-images/echoserver:2.2.
3. Подключиться локально к Pod с помощью `kubectl port-forward` и вывести значение (curl или в браузере).

### Решение
1. Создал манифест [hello-world.yaml](hello-world.yaml).
2. Активировал создание Pod:
    ```bash
    kubectl apply -f hello-world.yaml
    ```
    ``` 
    pod/hello-world created
    ```
3. Проверил состояние Pod:
    ```
    kubectl get pods
    ```
    ```
    NAME          READY   STATUS    RESTARTS   AGE
    hello-world   1/1     Running   0          17s
    ```
4. Подключился к Pod:
    ```
    kubectl port-forward pod/hello-world 8080
    ```
    ```
    curl localhost:8080
    ```
   ```
   Hostname: hello-world
   
   Pod Information:
       -no pod information available-
   
   Server values:
       server_version=nginx: 1.12.2 - lua: 10010
   
   Request Information:
       client_address=127.0.0.1
       method=GET
       real path=/
       query=
       request_version=1.1
       request_scheme=http
       request_uri=http://localhost:8080/
   
   Request Headers:
       accept=*/*  
       host=localhost:8080  
       user-agent=curl/7.81.0  
   
   Request Body:
       -no body in request-
   ```
------

## Задание 2. Создать Service и подключить его к Pod

1. Создать Pod с именем netology-web.
2. Использовать image — gcr.io/kubernetes-e2e-test-images/echoserver:2.2.
3. Создать Service с именем netology-svc и подключить к netology-web.
4. Подключиться локально к Service с помощью `kubectl port-forward` и вывести значение (curl или в браузере).

### Решение

1. Создал манифест для Pod [netology-web.yaml](netology-web.yaml).
2. Активировал создание Pod:
    ```bash
    kubectl apply -f netology-web.yaml
    ```
    ``` 
    pod/netology-web created
    ```
3. Проверил состояние Pod:
    ```
    kubectl get pods
    ```
    ```
    NAME           READY   STATUS    RESTARTS   AGE
    hello-world    1/1     Running   0          82m
    netology-web   1/1     Running   0          9s
    ```
2. Создал манифест для Service [netology-svc.yaml](netology-svc.yaml).
3. Активировал создание Service:
    ```bash
    kubectl apply -f netology-svc.yaml
    ```
    ``` 
    service/netology-svc created
    ```
4. Проверил состояние Service:
    ```
    kubectl get svc
    ```
    ```
    NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    kubernetes     ClusterIP   10.152.183.1     <none>        443/TCP    92m
    netology-svc   ClusterIP   10.152.183.204   <none>        8080/TCP   2m8s
    ```
5. Подключился к Service:
    ```
    kubectl port-forward svc/netology-svc 8080
    ```
    ```
    curl localhost:8080
    ```
   ```
   Hostname: netology-web
   
   Pod Information:
       -no pod information available-
      
   Server values:
       server_version=nginx: 1.12.2 - lua: 10010
      
   Request Information:
       client_address=127.0.0.1
       method=GET
       real path=/
       query=
       request_version=1.1
       request_scheme=http
       request_uri=http://localhost:8080/
      
   Request Headers:
       accept=*/*  
       host=localhost:8080  
       user-agent=curl/7.81.0  
      
   Request Body:
       -no body in request-
   
   ```