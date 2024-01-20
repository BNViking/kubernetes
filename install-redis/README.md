[[Перейти в начало](../README.md)]

---

## Установка Redis

* [Artifact Hub](https://artifacthub.io/packages/helm/bitnami/redis)
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

3. Создаем файл [./00-redis-values.yaml](./00-redis-values.yaml) с настройками
   ```bash
   helm show values bitnami/redis > ./00-redis-values.yaml
   ```

4. Редактируем настройки [./00-redis-values.yaml](./00-redis-values.yaml)
   ```yaml
   global:
      storageClass: "nfs-bnvkube-client"
   architecture: standalone
   auth:
      enabled: true
      password: "redispassword"
   metrics:
      enabled: true
   ```

5. Устанавливаем
   ```bash
   helm install redis bitnami/redis -f ./00-redis-values.yaml --namespace redis --create-namespace
   ```

---

### Удаление
   ```bash
   helm uninstall redis -n redis
   ```

Смотрим какие остались тома
   ```bash
   kubectl get pvc -n redis
   ```
   ```
   NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS         VOLUMEATTRIBUTESCLASS   AGE
   redis-data-redis-master-0   Bound    pvc-a061f894-5bc0-40d6-9b8b-e086f2e7d593   8Gi        RWO            nfs-bnvkube-client   <unset>                 21h
   ```

Удаляем том
   ```bash
   kubectl delete pvc -n redis redis-data-redis-master-0
   ```

---

[[Перейти в начало](../README.md)]