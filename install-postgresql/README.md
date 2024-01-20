[[Перейти в начало](../README.md)]

---

## Установка PostgreSql

* [Artifact Hub](https://artifacthub.io/packages/helm/bitnami/postgresql)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

### Пошаговая установка

1. Добавьте репозиторий и выберите версию для установки
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```

2. Обновляем кеш приложений (charts)
   ```bash
   helm repo update
   ```
3. Создадим namespace
   ```bash
   kubectl create namespace postgresql
   ```

4. Создадим секрет для авторизации в PostgreSql
   ```bash
   kubectl create secret generic postgresql-auth -n postgresql --from-literal=AdminPwd=adminpassword --from-literal=UserPwd=userpassword --from-literal=ReplicationPwd=replicapassword
   ```

5. Создаем файл [./00-postgresql-values.yaml](./00-postgresql-values.yaml) с настройками
   ```bash
   helm show values bitnami/postgresql > ./00-postgresql-values.yaml
   ```

6. Редактируем настройки [./00-postgresql-values.yaml](./00-postgresql-values.yaml)
   ```yaml
   global:
     storageClass: "nfs-bnvkube-client"
     postgresql:
       auth:
         database: "bnvdb"
       existingSecret: "postgresql-auth"
       secretKeys:
         adminPasswordKey: "AdminPwd"
         userPasswordKey: "UserPwd"
         replicationPasswordKey: "ReplicationPwd"
   ```

7. Устанавливаем
   ```bash
   helm install postgresql bitnami/postgresql -f ./00-postgresql-values.yaml --namespace postgresql --create-namespace
   ```

---

### Удаление
   ```bash
   helm uninstall postgresql -n postgresql
   ```

Смотрим какие остались тома
   ```bash
   kubectl get pvc -n postgresql
   ```
   ```
   NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS         VOLUMEATTRIBUTESCLASS   AGE
   data-postgresql-0   Bound    pvc-78f17bc2-d94f-4875-af41-95c60d722d15   8Gi        RWO            nfs-bnvkube-client   <unset>                 9m22s
   ```

Удаляем том
   ```bash
   kubectl delete pvc -n postgresql data-postgresql-0
   ```

---

[[Перейти в начало](../README.md)]