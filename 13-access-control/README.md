# Домашнее задание к занятию «Управление доступом»

------

### Задание 1. Создайте конфигурацию для подключения пользователя

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.
2. Настройте конфигурационный файл kubectl для подключения.
3. Создайте роли и все необходимые настройки для пользователя.
4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).
5. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.

------

1. Создал закрытый ключ:
    ```
    openssl genrsa -out user1.key 2048
    ```
2. Создал запрос на подпись сертификата: 
    ```
    openssl req -new -key user1.key -out user1.csr -subj "/CN=user1"
    ```
3. Подписал сертификат в Kubernetes CA:
    ```
    openssl x509 -req -in user1.csr -CA /var/snap/microk8s/current/certs/ca.crt -CAkey /var/snap/microk8s/current/certs/ca.key -CAcreateserial -out user1.crt -days 500
    ```
    ```
    Certificate request self-signature ok
    subject=CN = user1
    ```
4. Создал пользователя user1:
    ```
    kubectl config set-credentials user1 --client-certificate=user1.crt --client-key=user1.key
    ```
    ```
    User "user1" set.
    ```
5. Задал контекст для пользователя:
    ```
    kubectl config set-context user1-context --cluster=microk8s-cluster --user=user1
    ```
    ```
    Context "user1-context" created.
    ```
    ```
    kubectl config view
    ```
    ```
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: DATA+OMITTED
        server: https://192.168.1.109:16443
      name: microk8s-cluster
    contexts:
    - context:
        cluster: microk8s-cluster
        user: admin
      name: microk8s
    - context:
        cluster: microk8s-cluster
        user: user1
      name: user1-context
    current-context: microk8s
    kind: Config
    preferences: {}
    users:
    - name: admin
      user:
        token: REDACTED
    - name: user1
      user:
        client-certificate: /home/alex/repo/netology-devkub/13-access-control/cert/user1.crt
        client-key: /home/alex/repo/netology-devkub/13-access-control/cert/user1.key
    ```
4. Создал Role [role1.yaml](role1.yaml) и RoleBinding [rolebinding1.yaml](rolebinding1.yaml) для пользователя user1:
    ```
    kubectl apply -f role1.yaml
    ``` 
    ```
    role.rbac.authorization.k8s.io/pod-reader created
    ```
    ```
    kubectl apply -f rolebinding1.yaml
    ``` 
    ```
    rolebinding.rbac.authorization.k8s.io/read-pods created
    ```
5. Создал тестовый под [hello-world.yaml](hello-world.yaml):
    ```
    kubectl apply -f hello-world.yaml
    ``` 
    ```
    pod/hello-world created
    ```
6. Переключился на контекст user1-context:
    ```
    kubectl config use-context user1-context
    ```
    ```
    Switched to context "user1-context".
    ```
7. Проверил права пользователя - просмотр подов разрешён, удаление запрещено:
   ```
   kubectl get pods
   ```
   ```
   NAME          READY   STATUS    RESTARTS   AGE
   hello-world   1/1     Running   0          73s
   ```
   ```
   kubectl logs pods/hello-world
   ```
   ```
   Generating self-signed cert
   Generating a 2048 bit RSA private key
   ....+++
   ...............+++
   writing new private key to '/certs/privateKey.key'
   -----
   Starting nginx
   ```
   ```
   kubectl describe pods/hello-world 
   ```
   ```
   Name:             hello-world
   Namespace:        default
   Priority:         0
   Service Account:  default
   ...
   ```
   ```
   kubectl delete pod/hello-world
   ``` 
   ```
   Error from server (Forbidden): pods "hello-world" is forbidden: User "user1" cannot delete resource "pods" in API group "" in the namespace "default"
   ```
