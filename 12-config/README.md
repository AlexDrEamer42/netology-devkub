# Домашнее задание к занятию «Конфигурация приложений»

------

### Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Решить возникшую проблему с помощью ConfigMap.
3. Продемонстрировать, что pod стартовал и оба конейнера работают.
4. Сделать простую веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

1. Создал Deployment [deployment1.yaml](deployment1.yaml) с контейнерами, работающими на разных портах. Для multitool номер порта передал через ConfigMap [cm1.yaml](cm1.yaml):
   ```
   kubectl apply -f cm1.yaml 
   kubectl apply -f deployment1.yaml 
   ```
2. Проверил работу pod и контейнеров:
   ```
   kubectl get pods
   ```
   ```
   NAME                                READY   STATUS    RESTARTS   AGE
   nginx-deployment-5b44757fd4-69tdb   2/2     Running   0          14s
   ```
   ```
   kubectl exec -it nginx-deployment-5b44757fd4-69tdb -- bash
   ```
   ```
   Defaulted container "nginx" out of: nginx, multitool
   root@nginx-deployment-5b44757fd4-69tdb:/# curl localhost:80
   <html>
   <head><title>403 Forbidden</title></head>
   <body>
   <center><h1>403 Forbidden</h1></center>
   <hr><center>nginx/1.25.2</center>
   </body>
   </html>
   root@nginx-deployment-5b44757fd4-69tdb:/# curl localhost:8081
   WBITT Network MultiTool (with NGINX) - nginx-deployment-5b44757fd4-69tdb - 10.1.43.221 - HTTP: 8081 , HTTPS: 443 . (Formerly praqma/network-multitool)
   ```
3. Подключил к Nginx простую веб-страницу через ConfigMap. Подключил Service [nginx-service.yaml](nginx-service.yaml):
   ```
   kubectl apply -f nginx-service.yaml
   ``` 
   ```
   kubectl describe service/nginx-svc 
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
   IP:                10.152.183.62
   IPs:               10.152.183.62
   Port:              nginx  80/TCP
   TargetPort:        80/TCP
   Endpoints:         10.1.43.221:80
   Session Affinity:  None
   Events:            <none>
   ```
   ```
   curl 10.152.183.62:80/hello.html
   ```
   ```
   <html>
     Hello World!
   </html>
   ```
 
------

### Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS 

1. Создать Deployment приложения, состоящего из Nginx.
2. Создать собственную веб-страницу и подключить её как ConfigMap к приложению.
3. Выпустить самоподписной сертификат SSL. Создать Secret для использования сертификата.
4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

1. Создал Deployment [deployment2.yaml](deployment2.yaml):
   ```
   kubectl apply -f deployment2.yaml
   ```
2. Подключил веб-страницу через ConfigMap [cm2.yaml](cm2.yaml):
   ```
   kubectl apply -f cm2.yaml
   ```
3. Выпустил самоподписанный SSL-сертификат:
   ```
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
   ```
4. Закодировал файлы сертификата в base64:
   ```
   sudo cat /etc/ssl/private/apache-selfsigned.key | base64
   sudo cat /etc/ssl/certs/apache-selfsigned.crt | base64
   ```
5. Создал Secret с данными сертификата [secret2.yaml](secret2.yaml): 
   ```
   kubectl apply -f secret2.yaml
   ``` 
6. Создал Ingress [ingress2.yaml](ingress2.yaml) и необходимый Service [nginx-service2.yaml](nginx-service2.yaml): 
   ```
   kubectl apply -f nginx-service2.yaml 
   kubectl apply -f ingress2.yaml 
   ```
7. Сделал запись example.com для IP-адреса сервиса в /etc/hosts.
8. Проверил, что созданная страница открывается:
   ```
   curl https://example.com/hello.html
   ```
   ```
   <html>
     Hello World!
   </html>
   ```

