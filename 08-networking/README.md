# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Решение

1. Создал Deployment с манифестом [nginx_deployment1.yaml](nginx_deployment1.yaml):
    ```
    kubectl apply -f nginx_deployment1.yaml
    ``` 
    ```
    kubectl get pods
    ```
    ```
    NAME                                READY   STATUS    RESTARTS   AGE
    nginx-deployment-7d974d59c4-mncp7   2/2     Running   0          95s
    nginx-deployment-7d974d59c4-swg8m   2/2     Running   0          95s
    nginx-deployment-7d974d59c4-w6ctm   2/2     Running   0          95s
    ```
2. Создал Service с манифестом [nginx-service.yaml](nginx-service.yaml):
    ```
    kubectl apply -f nginx-service.yaml
    ``` 
    ```
    kubectl describe svc/nginx-svc
    ```
    ```
    Name:              nginx-svc
    Namespace:         default
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=nginx
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.152.183.155
    IPs:               10.152.183.155
    Port:              nginx  9001/TCP
    TargetPort:        80/TCP
    Endpoints:         10.1.43.241:80,10.1.43.242:80,10.1.43.243:80
    Port:              multitool  9002/TCP
    TargetPort:        8080/TCP
    Endpoints:         10.1.43.241:8080,10.1.43.242:8080,10.1.43.243:8080
    Session Affinity:  None
    Events:            <none>
    ```
3. Создал отдельный Pod с приложением multitool:  
    ```
    kubectl run multitool --image=wbitt/network-multitool
    ```
4. Убедился, что из пода есть доступ до приложения по разным портам в разные контейнеры.
    ```
    kubectl exec -it multitool /bin/bash
    ```
    ```
    multitool:/# curl 10.1.43.241:80 -I
    ```
    ```
    HTTP/1.1 200 OK
    Server: nginx/1.25.2
    Date: Thu, 21 Sep 2023 15:02:23 GMT
    Content-Type: text/html
    Content-Length: 615
    Last-Modified: Tue, 15 Aug 2023 17:03:04 GMT
    Connection: keep-alive
    ETag: "64dbafc8-267"
    Accept-Ranges: bytes
    ```
    ```
    multitool:/# curl 10.1.43.242:8080 -I
    ```
    ```
    HTTP/1.1 200 OK
    Server: nginx/1.24.0
    Date: Thu, 21 Sep 2023 15:02:29 GMT
    Content-Type: text/html
    Content-Length: 151
    Last-Modified: Thu, 21 Sep 2023 14:43:33 GMT
    Connection: keep-alive
    ETag: "650c5695-97"
    Accept-Ranges: bytes
    ```
5. Проверил доступ по доменному имени сервиса:
    ```
    multitool:/# curl nginx-svc:9002
    ```
    ```
    WBITT Network MultiTool (with NGINX) - nginx-deployment-7d974d59c4-w6ctm - 10.1.43.242 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
    multitool:/# curl nginx-svc:9001 -I
    HTTP/1.1 200 OK
    Server: nginx/1.25.2
    Date: Thu, 21 Sep 2023 14:58:23 GMT
    Content-Type: text/html
    Content-Length: 615
    Last-Modified: Tue, 15 Aug 2023 17:03:04 GMT
    Connection: keep-alive
    ETag: "64dbafc8-267"
    Accept-Ranges: bytes
    ```
    ```
    multitool:/# curl nginx-svc:9002 -I
    ```
    ```
    HTTP/1.1 200 OK
    Server: nginx/1.24.0
    Date: Thu, 21 Sep 2023 14:58:27 GMT
    Content-Type: text/html
    Content-Length: 151
    Last-Modified: Thu, 21 Sep 2023 14:43:30 GMT
    Connection: keep-alive
    ETag: "650c5692-97"
    Accept-Ranges: bytes
    ```
-----

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

------

1. Создал отдельный Service с манифестом [nginx-service2.yaml](nginx-service2.yaml) с возможностью доступа снаружи кластера через NodePort:
    ```
    kubectl apply -f nginx-service2.yaml
    ``` 
    ```
    kubectl get svc
    ```
    ```
    NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                       AGE
    kubernetes   ClusterIP   10.152.183.1    <none>        443/TCP                       10d
    nginx-svc    NodePort    10.152.183.71   <none>        80:30080/TCP,8080:30880/TCP   16s
    ```
2. Проверил доступ с помощью curl с локального компьютера:
    ```
    curl localhost:30080 -I
    ```
    ```
    HTTP/1.1 200 OK
    Server: nginx/1.25.2
    Date: Thu, 21 Sep 2023 15:08:51 GMT
    Content-Type: text/html
    Content-Length: 615
    Last-Modified: Tue, 15 Aug 2023 17:03:04 GMT
    Connection: keep-alive
    ETag: "64dbafc8-267"
    Accept-Ranges: bytes
    ```
    ```
    curl localhost:30880 -I
    ```
    ```
    HTTP/1.1 200 OK
    Server: nginx/1.24.0
    Date: Thu, 21 Sep 2023 15:08:56 GMT
    Content-Type: text/html
    Content-Length: 151
    Last-Modified: Thu, 21 Sep 2023 14:43:30 GMT
    Connection: keep-alive
    ETag: "650c5692-97"
    Accept-Ranges: bytes
    ```



