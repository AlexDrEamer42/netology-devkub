# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.
2. Создать Deployment приложения _backend_ из образа multitool. 
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

1. Создал Deployment приложения _frontend_: [frontend_deployment.yaml](frontend_deployment.yaml):
    ```
    kubectl apply -f frontend_deployment.yaml
    ``` 

2. Создал Deployment приложения _backend_: [backend_deployment.yaml](backend_deployment.yaml):
    ```
    kubectl apply -f backend_deployment.yaml
    ```

3. Создал сервисы [frontend_service.yaml](frontend_service.yaml) и [backend_service.yaml](backend_service.yaml) для доступа к приложениям:
    ```
    kubectl apply -f frontend_service.yaml
    kubectl apply -f backend_service.yaml 
    ```

4. Проверил список созданных подов и сервисов:
    ```
    kubectl get pods -o wide
    ```
    ```
    NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE      NOMINATED NODE   READINESS GATES
    frontend-bcc4d676-pvx7l    1/1     Running   0          43m   10.1.43.207   example   <none>           <none>
    frontend-bcc4d676-rgg95    1/1     Running   0          43m   10.1.43.205   example   <none>           <none>
    frontend-bcc4d676-hn75n    1/1     Running   0          43m   10.1.43.206   example   <none>           <none>
    backend-59556dfbcb-895k8   1/1     Running   0          43m   10.1.43.208   example   <none>           <none>
    ```
    ```
    kubectl get svc
    ```
    ```
    NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.152.183.1    <none>        443/TCP   14d
    frontend     ClusterIP   10.152.183.57   <none>        80/TCP    23h
    backend      ClusterIP   10.152.183.67   <none>        80/TCP    23h
    ```
5. Убедился, что приложения видят друг друга с помощью сервисов:
    ```
    kubectl exec --stdin --tty frontend-bcc4d676-pvx7l -- /bin/bash
    root@frontend-bcc4d676-pvx7l:/# curl backend
    ```
    ```
    WBITT Network MultiTool (with NGINX) - backend-59556dfbcb-895k8 - 10.1.43.208 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
    ```
    ```
    kubectl exec --stdin --tty backend-59556dfbcb-895k8  -- /bin/bash
    backend-59556dfbcb-895k8:/# curl frontend 
    ```
    ```
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    ...
    ```

------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.
2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
4. Предоставить манифесты и скриншоты или вывод команды п.2.

------

1. Включил Ingress-контроллер:
    ```
    microk8s enable ingress
    ```
2. Добавил в **/etc/hosts** запись my-cluster.com с локальным IP-адресом кластера.  
3. Создал ingress: [ingress.yaml](ingress.yaml):
    ```
    kubectl apply -f ingress.yaml 
    kubectl describe ingress minimal-ingress
    ```
    ```
    Name:             minimal-ingress
    Labels:           <none>
    Namespace:        default
    Address:          127.0.0.1
    Ingress Class:    nginx
    Default backend:  <default>
    Rules:
      Host            Path  Backends
      ----            ----  --------
      my-cluster.com  
                      /      frontend:80 (10.1.43.205:80,10.1.43.206:80,10.1.43.207:80)
                      /api   backend:80 (10.1.43.208:80)
    Annotations:      nginx.ingress.kubernetes.io/rewrite-target: /
    Events:
      Type    Reason  Age                   From                      Message
      ----    ------  ----                  ----                      -------
      Normal  Sync    2m58s (x51 over 28m)  nginx-ingress-controller  Scheduled for sync
      Normal  Sync    2m58s (x51 over 28m)  nginx-ingress-controller  Scheduled for sync
    ```
4. Проверил доступ к приложениям через Ingress:
    ```
    curl -k https://my-cluster.com
    ```
    ```
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    ```
    ```
    curl -k https://my-cluster.com/api
    ...
    ```
    ```
    WBITT Network MultiTool (with NGINX) - backend-59556dfbcb-895k8 - 10.1.43.208 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
    ```