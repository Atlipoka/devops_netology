### Создать Deployment приложения, использующего локальный PV, созданный вручную.
***
1. Создаем Deployment приложения, состоящего из контейнеров busybox и multitool.
```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: volumes
  labels:
    app: vol
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vol
  template:
    metadata:
      labels:
        app: vol
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
          name: http-port
        volumeMounts:
        - name: vol
          mountPath: /output
      - name: busybox
        image: busybox:1.28
        command: ['sh', '-c','while true; do echo "Текущая дата и время" `date` >> /input/log.txt; sleep 5; done']
        volumeMounts:
        - name: vol
          mountPath: /input
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: pvc
      restartPolicy: Always
```
2. Создаем PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
  namespace: default
spec:
  storageClassName: local
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local
  local:
    path: "/home/vagrant/kubernetes/kuber_Storing2"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - vagrant
```
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории.
````
vagrant@vagrant:~/kubernetes/kuber_Storing2$ kubectl exec -i -t volumes-54c9787bc8-t2586 --container multitool -- /bin/bash
bash-5.1# tail -n 10 output/log.txt
Текущая дата и время Thu Aug 10 14:16:13 UTC 2023
Текущая дата и время Thu Aug 10 14:16:13 UTC 2023
Текущая дата и время Thu Aug 10 14:16:18 UTC 2023
Текущая дата и время Thu Aug 10 14:16:18 UTC 2023
Текущая дата и время Thu Aug 10 14:16:23 UTC 2023
Текущая дата и время Thu Aug 10 14:16:23 UTC 2023
Текущая дата и время Thu Aug 10 14:16:28 UTC 2023
Текущая дата и время Thu Aug 10 14:16:28 UTC 2023
Текущая дата и время Thu Aug 10 14:16:33 UTC 2023
Текущая дата и время Thu Aug 10 14:16:33 UTC 2023
````
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
 - После удаления deployment и pvc объект pv перешел в статус Released - это означает, что том свободен и удалить его необходимо будет вручную. Это произошло потому, что при создании тома была указана политика Retain. Потворно использовать pv не удасться, т.к. мы указали политику доступа ReadWriteOnce, что означает одноразовое использование тома с одни ресуросом.
````
vagrant@vagrant:~/kubernetes/kuber_Storing2$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                               STORAGECLASS        REASON   AGE
pvc-3dc219aa-7b16-4b4a-8585-ac26387864dd   20Gi       RWX            Delete           Bound      container-registry/registry-claim   microk8s-hostpath            10d
pv                                         1Gi        RWO            Retain           Released   default/pvc                         local                       5m4s
````
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV. Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
 - После удаления pv файл отсался по пути и не удалился, остался, как и было написано выше, в этом Retain
````
agrant@vagrant:~/kubernetes/kuber_Storing2$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                               STORAGECLASS        REASON   AGE
pvc-3dc219aa-7b16-4b4a-8585-ac26387864dd   20Gi       RWX            Delete           Bound    container-registry/registry-claim   microk8s-hostpath            10d
vagrant@vagrant:~/kubernetes/kuber_Storing2$ tail -n 10 log.txt
Текущая дата и время Thu Aug 10 14:42:23 UTC 2023
Текущая дата и время Thu Aug 10 14:42:24 UTC 2023
Текущая дата и время Thu Aug 10 14:42:28 UTC 2023
Текущая дата и время Thu Aug 10 14:42:29 UTC 2023
Текущая дата и время Thu Aug 10 14:42:33 UTC 2023
Текущая дата и время Thu Aug 10 14:42:34 UTC 2023
Текущая дата и время Thu Aug 10 14:42:38 UTC 2023
Текущая дата и время Thu Aug 10 14:42:39 UTC 2023
Текущая дата и время Thu Aug 10 14:42:43 UTC 2023
Текущая дата и время Thu Aug 10 14:42:44 UTC 2023
````

### Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.
***
1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода.
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.
