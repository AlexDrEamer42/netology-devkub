# Домашнее задание к занятию «Хранение в K8s. Часть 2»

------

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

1. Включил использование класса hostpath в MicroK8s:
    ```
    microk8s enable hostpath-storage
    ```
2. Создал Deployment [deployment1.yaml](deployment1.yaml), PV [pv1.yaml](pv1.yaml), PVC [pvc1.yaml](pvc1.yaml):
    ```
    kubectl apply -f pv1.yaml 
    kubectl apply -f pvc1.yaml 
    kubectl apply -f deployment1.yaml
    ``` 
3. Убедился, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории:
    ```
    kubectl exec -it volume-pod-76bf798c5-wlr8k -c multitool -- cat /input/some_file.txt
    ```
    ```
    Sat Oct  7 06:59:17 UTC 2023
    Sat Oct  7 06:59:22 UTC 2023
    Sat Oct  7 06:59:27 UTC 2023
    Sat Oct  7 06:59:32 UTC 2023
    Sat Oct  7 06:59:37 UTC 2023
    ````
4. Удалил Deployment и PVC:
    ```
    kubectl delete deployments.apps volume-pod 
    kubectl delete pvc task-pv-claim
    ``` 
5. Проверил состояние PV:
    ```
    kubectl get pv
    ```
    ```
    NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                   STORAGECLASS   REASON   AGE
    task-pv-volume   10Gi       RWO            Retain           Released   default/task-pv-claim   manual                  3m48s
    ```
6. Убедился, что файл сохранился на локальном диске ноды:
    ```
    ls -lah /mnt/data
    ```
    ```
    итого 12K
    drwxr-xr-x 2 root root 4,0K окт  7 14:59 .
    drwxr-xr-x 3 root root 4,0K окт  7 14:59 ..
    -rw-r--r-- 1 root root  986 окт  7 15:02 some_file.txt
    ```
7. Удалил PV:
    ```
    kubectl delete pv task-pv-volume
    ``` 
8. Проверил состояние файла:
    ```
    ls -lah /mnt/data
    ```
    ```
    итого 12K
    drwxr-xr-x 2 root root 4,0K окт  7 14:59 .
    drwxr-xr-x 3 root root 4,0K окт  7 14:59 ..
    -rw-r--r-- 1 root root  986 окт  7 15:02 some_file.txt
    ```
    Ресурсы остались, потому что по умолчанию параметр `persistentVolumeReclaimPolicy` для PV имеет значение `Retain` (не удалять созданные ресурсы).

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

1. Установил и настроил NFS-сервер по инструкции: https://microk8s.io/docs/nfs
2. Создал Storage class [sc-nfs.yaml](sc-nfs.yaml) и PVC [pvc-nfs.yaml](pvc-nfs.yaml).
3. Проверил статус PVC:
    ```
    kubectl get pvc
    ```
    ```
    NAME     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    my-pvc   Bound    pvc-14fcec27-a39b-47b4-bfac-b8faaefc9c3b   5Gi        RWO            nfs-csi        17s
    ```
4. Создал Deployment [deployment2.yaml](deployment2.yaml):   
    ```
    kubectl apply -f deployment2.yaml 
    ```
5. Проверил возможность чтения и записи файла изнутри пода:   
    ```
    kubectl exec -it volume-pod-6b5b77c485-m85kr -c multitool -- bash
    volume-pod-6b5b77c485-m85kr:/# touch /input/some_file.txt
    volume-pod-6b5b77c485-m85kr:/# echo 123 > /input/some_file.txt 
    volume-pod-6b5b77c485-m85kr:/# cat /input/some_file.txt 
    123
    ```
6. Проверил содержимое файла на ноде:
    ```
    cat /srv/nfs/pvc-14fcec27-a39b-47b4-bfac-b8faaefc9c3b/some_file.txt 
    123
    ```



