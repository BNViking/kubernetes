[[Перейти в начало](../README.md)]

---

## Установка PgAdmin 4

* [pgAdmin](https://www.pgadmin.org/)
* [Установка PostgreSql](../install-postgresql/README.md)
* [Artifact Hub](https://artifacthub.io/packages/helm/runix/pgadmin4)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

### Пошаговая установка

1. Добавьте репозиторий и выберите версию для установки
   ```bash
   helm repo add runix https://helm.runix.net
   ```

2. Обновляем кеш приложений (charts)
   ```bash
   helm repo update
   ```

3. Создаем файл [./00-pgadmin4-values.yaml](./00-pgadmin4-values.yaml) с настройками
   ```bash
   helm show values runix/pgadmin4 > ./00-pgadmin4-values.yaml
   ```

4. Редактируем настройки [./00-pgadmin4-values.yaml](./00-pgadmin4-values.yaml)
   ```yaml
   ingress:
      enabled: true
      annotations:
         cert-manager.io/cluster-issuer: self-signed
      ingressClassName: "nginx"
      hosts:
         - host: pgadmin.bnvkube.lan
      tls:
         - secretName: pgadmin-tls
           hosts:
              - pgadmin.bnvkube.lan
   env:
      email: admin@bnvkube.lan
      password: pgadminpassword
   persistentVolume:
      storageClass: "nfs-bnvkube-client"
   ```

5. Устанавливаем
   ```bash
   helm install pgadmin4 runix/pgadmin4 -f ./00-pgadmin4-values.yaml --namespace postgresql --create-namespace
   ```

---

### Удаление
   ```bash
   helm uninstall pgadmin4 -n postgresql
   ```

---

[[Перейти в начало](../README.md)]