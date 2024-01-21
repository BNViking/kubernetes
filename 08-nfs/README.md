[[Перейти в начало](../README.md)]

---

## Настройка NFS

* [Добавление нового диска](#add-new-disk)
* [Создание сервера NFS](#install-nfs-server)
* [Настройка автоматического создание раздела NFS в Kubernetes](#auto-nfs-kube)

*Если у вас есть созданный раздел NFS, то "Добавление нового диска" и "Создание сервера NFS" можно пропустить*

---

### <a name="add-new-disk"></a>Добавление нового диска
Выбираем любой узел, который будет выступать в качестве сервера NFS.

1. Добавить диск на машине и прикрепляем его

2. Смотрим диск в системе
    ```bash
    sudo fdisk -l
    ```
    ```
    ...
    Disk /dev/sdb: 110 GiB, 118111600640 bytes, 230686720 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    ```

3. Создаем разметку gpt и раздел диска с помощью утилиты **fdisk** (Команды: g, потом n)
    ```bash
    sudo fdisk /dev/sdb
    ```

4. Проверяем созданный раздел
    ```bash
    lsblk
    ```
    ```
    ...
    sdb      8:16   0   110G  0 disk
    └─sdb1   8:17   0   110G  0 part
    ```

5. Отформатируем файловую систему в ext4 (есть ограничение на количество файлов в одной папке)\
   ```bash
    sudo mkfs.ext4 /dev/sdb1
    ```

6. Создаем папку для монтирования
    ```bash
    sudo mkdir -p /var/nfs/bnvkube
    ```

7. Смотрит UUID диска
    ```bash
    ll /dev/disk/by-uuid/
    ```
    ```
    lrwxrwxrwx 1 root root  10 янв 16 20:59 6ee1d812-3bb0-475b-8593-1aeef112096c -> ../../sdb1
    ```

8. Добавим запись в /etc/fstab, что бы диск автоматически монтировался при загрузке
    ```
    /dev/disk/by-uuid/6ee1d812-3bb0-475b-8593-1aeef112096c /var/nfs/bnvkube ext4 defaults 0 1
    ```

9. Перемонтируем разделы
    ```bash
    sudo mount -a
    ```

10. Проверяем монтирование раздела
   ```bash
   df -h
   ```
   ```
   Filesystem      Size  Used Avail Use% Mounted on
   tmpfs           392M  2,2M  390M   1% /run
   ...
   /dev/sdb1       108G   24K  103G   1% /var/nfs/bnvkube
   ```

11. Меняем владельца и группу
     ```bash
     sudo chown -R nobody:nogroup /var/nfs/bnvkube
     ```
   
12. Проверяем права
   ```bash
   ls -la /var/nfs/bnvkube
   ```
   ```
   total 24
   drwxr-xr-x 3 nobody nogroup  4096 янв 16 20:59 .
   drwxr-xr-x 3 root   root     4096 янв 16 20:52 ..
   drwx------ 2 nobody nogroup 16384 янв 16 20:59 lost+found
   ```

---

### <a name="install-nfs-server"></a> Создание сервера NFS

1. Ставим пакеты
   ```bash
   sudo apt-get update && sudo apt-get install nfs-kernel-server -y
   ```

2. Редактируем файл /etc/exports, добавляем запись для нашего раздела (Доступ для сети 192.168.1.0/24)
   ```bash
   /var/nfs/bnvkube     192.168.1.0/24(rw,sync,no_subtree_check,insecure,no_root_squash)
   ```

3. Применяем настройки nfs
   ```bash
   sudo exportfs -ar
   ```

4. Проверяем
   ```bash
   sudo exportfs -v
   ```
   ```
   /var/nfs/bnvkube
       192.168.1.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,insecure,no_root_squash,no_all_squash)
   ```

---
Если не выключен firewall, то необходимо добавить разрешения
   ```bash
   sudo ufw allow 2049
   sudo ufw allow 111
   sudo ufw allow from 192.168.1.0/24 to any port nfs
   ```
---

#### Описание опций монтирования каталогов NFS

В файле **/etc/exports** используются следующие общие опции\
_(сначала указаны опции применяемые по-умолчанию в большинстве систем,
в скобках - не по-умолчанию)_:

**auth_nlm** (**no_auth_nlm**) или **secure_locks** (**insecure_locks**) - указывает, что сервер должен требовать 
аутентификацию запросов на блокировку (с помощью протокола NFS Lock Manager (диспетчер блокировок NFS)).

**nohide** (**hide**) - если сервер экспортирует две иерархии каталогов, при этом одна вложенна (примонтированна) в другую. 
Клиенту необходимо явно смонтировать вторую (дочернюю) иерархию, иначе точка монтирования дочерней иерархии будет выглядеть как пустой каталог. 
Опция **nohide** приводит к появлению второй иерархии каталогов без явного монтирования.

**ro** (**rw**) - Разрешает только запросы на чтение (запись).\
_(возможно прочитать/записать определяется на основании прав файловой системы, при этом сервер не способен отличить запрос 
на чтение файла от запроса на исполнение, поэтому разрешает чтение, если у пользователя есть права на чтение или исполнение.)_

**secure** (**insecure**) - требует, чтобы запросы NFS поступали с защищенных портов (< 1024), чтобы программа без прав root не могла монтировать иерархию каталогов.

**subtree_check** (**no_subtree_check**) - Если экспортируется подкаталог файловой системы, но не вся файловая система, 
сервер проверяет, находится ли запрошенный файл в экспортированном подкаталоге.\
Отключение проверки уменьшает безопасность, но увеличивает скорость передачи данных.

**sync** (**async**) - указывает, что сервер должен отвечать на запросы только после записи на диск изменений, 
выполненных этими запросами.\
_Опция **async** указывает серверу не ждать записи информации на диск, что повышает производительность, 
но понижает надежность, т.к. в случае обрыва соединения или отказа оборудования возможна потеря информации._

**wdelay** (**no_wdelay**) - указывает серверу задерживать выполнение запросов на запись, если ожидается последующий запрос на запись, 
записывая данные более большими блоками. Это повышает производительность при отправке больших очередей команд на запись.  
**no_wdelay** указывает не откладывать выполнение команды на запись, что может быть полезно, если сервер получает большое количество команд не связанных друг с другом.

**root_squash** (**no_root_squash**) - При заданной опции **root_squash**, запросы от пользователя root отображаются на 
анонимного uid/gid, либо на пользователя, заданного в параметре anonuid/anongid.

**no_all_squash** (**all_squash**) - Не изменяет UID/GID подключающегося пользователя.\
Опция **all_squash** задает отображение ВСЕХ пользователей (не только root), как анонимных или заданных в параметре anonuid/anongid.

**anonuid**=UID и **anongid**=GID - Явно задает UID/GID для анонимного пользователя.\
_map_static=/etc/file_maps_users - Задает файл, в котором можно задать сопоставление удаленных UID/GID - локальным UID/GID._

---

### <a name="auto-nfs-kube"></a> Настройка автоматического создание раздела NFS в Kubernetes 

* [Установка Helm](../install-helm/README.md)
* [Install CSI driver with Helm 3](https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/charts/README.md)
* [NFS Subdirectory External Provisioner Helm Chart](https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner)
* [GitHub](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

---

#### Установка драйвера CSI для NFS

1. Добавляем репозиторий
   ```bash
   helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
   ```
   
2. Обновляем кеш приложений (charts)
   ```bash
   helm repo update
   ```

3. Устанавливаем
   ```bash
   helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system
   ```

---

#### Настройка автоматического создания томов

1. Добавляем репозиторий
   ```bash
   helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
   ```

2. Обновляем кеш приложений (charts)
   ```bash
   helm repo update
   ```

3. Создаем файл [./00-nfs-external-provisioner-value.yaml](./00-nfs-external-provisioner-value.yaml) для настройки 
   ```bash
   helm show values nfs-subdir-external-provisioner --repo https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/ > ./00-nfs-external-provisioner-value.yaml
   ```

4. Меняем настройки в файле [./00-nfs-external-provisioner-value.yaml](./00-nfs-external-provisioner-value.yaml)
   ```yaml
   nfs:
       server: 192.168.1.15
       path: /var/nfs/bnvkube
       volumeName: nfs-bnvkube-root
       #Retain/Delete - оставлять/удалять данные при удалении раздела 
       reclaimPolicy: Retain
   storageClass:
       name: nfs-bnvkube-client #Наименование класса
       archiveOnDelete: false #Не архивировать раздел (переименовывает папку в архивную)
       accessModes: ReadWriteMany
   ```
5. Устанавливаем
   ```bash
   helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -f ./00-nfs-external-provisioner-value.yaml --create-namespace --namespace nfs-external-provisioner
   ```
6. Проверяем созданный класс
   ```bash
   kubectl get storageclasses
   ```
   ```
   NAME                 PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
   nfs-bnvkube-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   4m7s
   ```

7. Создадим тестовый раздел **pvctest** pvc, пример файла [01-nfs-volume-claime.yaml](./01-nfs-volume-claime.yaml)
   ```yaml
   spec:
       storageClassName: nfs-bnvkube-client
   ```
   ```bash
   kubectl apply -f ./01-nfs-volume-claime.yaml
   ```

8. Проверяем созданный **pvc**
   ```bash
   kubectl get pvc -A
   ```
   ```
   NAMESPACE   NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS         VOLUMEATTRIBUTESCLASS   AGE
   default     pvctest   Bound    pvc-e0b6aa87-9168-4e2d-9e0b-eb1018cea256   100Mi      RWX            nfs-bnvkube-client   <unset>                 8s
   ```
   ***STATUS = Bound** - Значит все ОК*
   
9. Удаляем тестовый раздел **pvctest**
   ```bash
   kubectl delete -f ./01-nfs-volume-claime.yaml
   ```

---

Для того что бы автоматически создавались **pvc** необходимо указать созданный класс `nfs-bnvkube-client` в настройках манифеста
```yaml
spec:
  storageClassName: nfs-bnvkube-client
```


---

[[Перейти в начало](../README.md)]