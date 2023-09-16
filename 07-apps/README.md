# Домашнее задание к занятию «Запуск приложений в K8S»

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

------

1. Создал манифест [nginx_deployment1.yaml](nginx_deployment1.yaml).
2. Активировал создание Deployment:
    ```
    kubectl apply -f nginx_deployment1.yaml
    ```
3. Проверил создание Pods:
    ```
    kubectl get pods
    ```
    ```
    NAME                                READY   STATUS    RESTARTS   AGE
    nginx-deployment-7d974d59c4-2jmpt   2/2     Running   0          53s
    ```
4. Изменил значение параметра `replicas` в манифесте с 1 на 2.
5. Пересоздал Deployment:
    ```
    kubectl apply -f nginx_deployment1.yaml
    ```
6. Проверил состояние Pods:
    ```
    kubectl get pods
    ```
    ```
    NAME                                READY   STATUS    RESTARTS   AGE
    nginx-deployment-7d974d59c4-2jmpt   2/2     Running   0          2m28s
    nginx-deployment-7d974d59c4-fv2sl   2/2     Running   0          29s
    ```
7. Создал манифест для Service [nginx-service.yaml](nginx-service.yaml).
8. Активировал создание Service:
    ```
    kubectl apply -f nginx-service.yaml
    ```
9. Проверил состояние Service:
    ```
    kubectl get svc/nginx-svc 
    ```
    ```
    NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
    nginx-svc   ClusterIP   10.152.183.82   <none>        80/TCP,8080/TCP   27s
    ```
10. Создал отдельный Pod с multitool:
    ```
    kubectl run multitool --image=wbitt/network-multitool
    ```
11. Проверил состояние Pods и их IP-адреса:
    ```
    kubectl get pods -o wide
    ```
    ```
    NAME                                READY   STATUS    RESTARTS   AGE    IP            NODE      NOMINATED NODE   READINESS GATES
    nginx-deployment-7d974d59c4-2jmpt   2/2     Running   0          18m    10.1.43.221   example   <none>           <none>
    nginx-deployment-7d974d59c4-fv2sl   2/2     Running   0          16m    10.1.43.222   example   <none>           <none>
    multitool                           1/1     Running   0          114s   10.1.43.224   example   <none>           <none>
    ```
12. Подключился к Pod multitool и проверил состояние приложений:
    ```
    kubectl exec -it multitool /bin/bash
    ```
    ```
    curl 10.1.43.221
    ```
    ```
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    ...
    ```    
    ```    
    curl 10.1.43.221:8080
    ```
    ```
    WBITT Network MultiTool (with NGINX) - nginx-deployment-7d974d59c4-2jmpt - 10.1.43.221 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
    ```
    ```
    curl 10.1.43.222
    ```
    ```
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    ...
    ```    
    ```    
    curl 10.1.43.222:8080
    ```
    ```
    WBITT Network MultiTool (with NGINX) - nginx-deployment-7d974d59c4-fv2sl - 10.1.43.222 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
    ```
-----

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

------

1. Создал Deployment c init-контейнером nginx. Убедился, что nginx не стартует:
    ```
      initContainers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
    ```
2. Указал в качестве init-контейнера busybox: [nginx_deployment2.yaml](nginx_deployment2.yaml).
3. Активировал создание Deployment:
    ```
    kubectl apply -f nginx_deployment1.yaml
    ```
4. Проверил создание Pods:
    ```
    kubectl get pods
    ```
    Состояние до запуска сервиса:
    ```
    NAME                               READY   STATUS     RESTARTS   AGE
    nginx-deployment-cffdb74d7-7ks2z   0/1     Init:0/1   0          6s
    ```
    Состояние после запуска сервиса:
    ```
    NAME                               READY   STATUS    RESTARTS   AGE
    nginx-deployment-cffdb74d7-7ks2z   1/1     Running   0          38s
    ```
