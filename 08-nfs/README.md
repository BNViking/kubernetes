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
   /var/nfs/bnvkube     192.168.1.0/24(rw,sync,subtree_check,all_squash,root_squash)
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
                   <world>(sync,wdelay,hide,sec=sys,rw,secure,root_squash,all_squash)
   ```

---
Если не выключен firewall, то необходимо добавить разрешения
   ```bash
   sudo ufw allow 2049
   sudo ufw allow 111
   sudo ufw allow from 192.168.1.0/24 to any port nfs
   ```
---

### <a name="auto-nfs-kube"></a> Настройка автоматического создание раздела NFS в Kubernetes 

* [Установка Helm](../install-helm/README.md)
* [Install CSI driver with Helm 3](https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/charts/README.md)
* [NFS Subdirectory External Provisioner Helm Chart](https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner)

1. Устанавливаем драйвер CSI
   ```bash
   helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
   ```
   ```bash
   helm repo update
   ```
   ```bash
   helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system
   ```
   
2. Установка автоматического создания разделов
   ```bash
   helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
   ```
   ```bash
   helm repo update
   ```
   ```bash
   helm show values nfs-subdir-external-provisioner --repo https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/ > ./00-nfs-external-provisioner-value.yaml
   ```

3. Меняем настройки в файле [00-nfs-external-provisioner-value.yaml](./00-nfs-external-provisioner-value.yaml)
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
4. Устанавливаем
   ```bash
   helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -f ./00-nfs-external-provisioner-value.yaml --create-namespace --namespace nfs-external-provisioner
   ```
5. Проверяем созданный класс
   ```bash
   kubectl get storageclasses
   ```
   ```
   NAME                 PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
   nfs-bnvkube-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   4m7s
   ```

6. Создадим тестовый раздел **pvctest** pvc, пример файла [01-nfs-volume-claime.yaml](./01-nfs-volume-claime.yaml)
   ```yaml
   spec:
       storageClassName: nfs-bnvkube-client
   ```
   ```bash
   kubectl apply -f ./01-nfs-volume-claime.yaml
   ```

7. Проверяем
   ```bash
   kubectl get pvc -A
   ```
   ```
   NAMESPACE   NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS         VOLUMEATTRIBUTESCLASS   AGE
   default     pvctest   Bound    pvc-e0b6aa87-9168-4e2d-9e0b-eb1018cea256   100Mi      RWX            nfs-bnvkube-client   <unset>                 8s
   ```
   *STATUS = Bound - Значит все ОК*
   
8. Удаляем тестовый раздел **pvctest**
   ```bash
   kubectl delete -f ./01-nfs-volume-claime.yaml
   ```

---

[[Перейти в начало](../README.md)]