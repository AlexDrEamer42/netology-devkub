# Домашнее задание к занятию «Хранение в K8s. Часть 1»

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.
4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.

------

1. Создал Deployment с манифестом [deployment1.yaml](deployment1.yaml):
    ```
    kubectl apply -f deployment1.yaml
    ```
   Контейнер `busybox` каждые 5 секунд записывает в файл `some_file.txt` текущее время и дату. 
2. Проверил статус пода:
    ```
    kubectl get pods
    ```
    ```
    NAME                          READY   STATUS        RESTARTS   AGE
    volume-pod-6566dd7598-7h7pn   2/2     Running       0          13s
    ```
3. Проверил содержимое файла `some_file.txt` из контейнера `multitool`: 
    ```
    kubectl exec -it volume-pod-6566dd7598-7h7pn -c multitool -- cat /input/some_file.txt
    ```
    ```
    Sat Sep 30 03:23:24 UTC 2023
    Sat Sep 30 03:23:29 UTC 2023
    Sat Sep 30 03:23:34 UTC 2023
    Sat Sep 30 03:23:39 UTC 2023
    Sat Sep 30 03:23:44 UTC 2023
    Sat Sep 30 03:23:49 UTC 2023
    Sat Sep 30 03:23:54 UTC 2023
    ```
    Видим, что данные обновляются каждые 5 секунд.

------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.
3. Продемонстрировать возможность чтения файла изнутри пода.
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

------

1. Создал DaemonSet с манифестом [daemonset1.yaml](daemonset1.yaml):
    ```
    kubectl apply -f daemonset1.yaml
    ``` 
2. Проверил статус пода:
    ```
    kubectl get pods
    ```
    ```
    NAME         READY   STATUS    RESTARTS   AGE
    logs-8q6bf   1/1     Running   0          19s
    ```
3. Проверил доступ к `/var/log/syslog` из пода:
    ```
    kubectl exec -it logs-8q6bf -c multitool -- tail -n 10 /logs/syslog
    ```
    ```
    Sep 30 11:38:45 example systemd[1]: run-containerd-runc-k8s.io-b4de1fa4868cc96ef6e5a13927eabe1d5fc5c23b85ff8049ac0e192247aa50c7-runc.AuExAQ.mount: Deactivated successfully.
    Sep 30 11:38:50 example rtkit-daemon[1326]: Supervising 7 threads of 5 processes of 1 users.
    Sep 30 11:38:50 example rtkit-daemon[1326]: Supervising 7 threads of 5 processes of 1 users.
    Sep 30 11:38:50 example microk8s.daemon-kubelite[2340]: I0930 11:38:50.604542    2340 trace.go:219] Trace[348884640]: "GuaranteedUpdate etcd3" audit-id:,key:/masterleases/192.168.1.109,type:*v1.Endpoints,resource:apiServerIPInfo (30-Sep-2023 11:38:49.837) (total time: 654ms):
    Sep 30 11:38:50 example microk8s.daemon-kubelite[2340]: Trace[348884640]: ---"Txn call completed" 649ms (11:38:50.491)
    Sep 30 11:38:50 example microk8s.daemon-kubelite[2340]: Trace[348884640]: [654.83063ms] [654.83063ms] END
    Sep 30 11:38:51 example systemd[1]: run-containerd-runc-k8s.io-1baa80a4a852fed5566f21a64ee5d5f0962263e45d7ec591611c80e66a25bbe1-runc.tl7zpz.mount: Deactivated successfully.
    Sep 30 11:38:52 example systemd[1]: run-containerd-runc-k8s.io-1baa80a4a852fed5566f21a64ee5d5f0962263e45d7ec591611c80e66a25bbe1-runc.0qwSma.mount: Deactivated successfully.
    Sep 30 11:39:02 example systemd[1]: run-containerd-runc-k8s.io-1baa80a4a852fed5566f21a64ee5d5f0962263e45d7ec591611c80e66a25bbe1-runc.wfg99D.mount: Deactivated successfully.
    Sep 30 11:39:07 example systemd[1]: run-containerd-runc-k8s.io-b047c4bfd3971f78ed4babeaec9e2ab92d8ebb1a21c2f8370b7dee49e91148df-runc.r3zcX1.mount: Deactivated successfully.
    ```
    Видим, что доступ к файлу есть.