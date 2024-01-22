[[Перейти в начало](../README.md)]

---

## Установка GitLab

* [GitLab Chart](https://gitlab.com/gitlab-org/charts/gitlab)
* [GitLab документация](https://docs.gitlab.com/charts/)
* [Описание параметров](https://docs.gitlab.com/charts/charts/)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

---

Устанавливаем
* [NFS](../08-nfs/README.md)
* [Cert Manager](./install-cert-namager/README.md) + [Настройка](../09-cert-manager)
* [MetalLB](../install-metal-lb/README.md)
* [Ingress NGINX Controller](../install-ingress-nginx-controller/README.md)
* [Prometheus Stack](../install-prometheus-stack/README.md)
* [Redis](../install-redis/README.md)
* [PostgreSql](../install-postgresql/README.md)
* [Minio](../install-minio/README.md)

_В **Minio** создать ключ для **GitLab**_\
_После создания **Access Key** в **minio**, сохраните файл **credentials.json**_
```json
{
   "url":"https://minio.bnvkube.lan/api/v1/service-account-credentials",
   "accessKey":"8LJljPcf89RAkIg3IpFk",
   "secretKey":"yfm55YsFJxJwMNWfQMExcnL5SNfl8scBeiYXQKYq",
   "api":"s3v4",
   "path":"auto"
}
```
_Создать базу данных **gitlab** в **PostgreSql**_ ([PgAdmin 4](../install-pgAdmin4/README.md))

---

### Пошаговая установка

1. Добавьте репозиторий и выберите версию для установки
   ```bash
   helm repo add gitlab https://charts.gitlab.io/
   ```

2. Обновляем кеш приложений (charts)
   ```bash
   helm repo update
   ```

3. Список доступных версий
   ```bash
   helm search repo gitlab -l
   ```

4. Создадим namespace
   ```bash
   kubectl create namespace gitlab
   ```

5. Создадим секретный ключ для подключения к **PostgreSql**
   ```bash
   kubectl create secret generic gitlab-postgresql-password -n gitlab --from-literal=postgres-password=userpassword
   ```

6. Создадим секретный ключ для подключения к **Redis**
   ```bash
   kubectl create secret generic gitlab-redis-password -n gitlab --from-literal=redis-password=redispassword
   ```
   
7. Создадим секретный ключ для GitLab
   ```bash
   kubectl create secret generic gitlab-root-password -n gitlab --from-literal=password=$(head -c 512 /dev/urandom | tr -cd 'a-zA-Z0-9' | head -c 16)
   ```

10. Создаем файл [./00-gitlab-values.yaml](./00-gitlab-values.yaml) с настройками
    ```bash
    helm show values gitlab/gitlab > ./01-gitlab-values.yaml
    ```

11. Редактируем настройки [./00-gitlab-values.yaml](./00-gitlab-values.yaml)
    ```yaml
    global:
      edition: ce
      hosts:
        domain: bnvkube.lan
      ingress:
        configureCertmanager: false
        class: nginx
        annotations:
          cert-manager.io/cluster-issuer: self-signed
        enabled: true
        tls:
          enabled: true
          secretName: gitlab-tls
        path: /
        pathType: Prefix
      monitoring:
        enabled: true
      initialRootPassword:
        secret: gitlab-root-password
        key: password
      psql:
        password:
          secret: gitlab-postgresql-password
          key: postgres-password
        host: postgresql.postgresql.svc.cluster.local
        port: 5432
        username: bnviking
        database: gitlab
      redis:
        auth:
          enabled: true
          secret: gitlab-redis-password
          key: redis-password
        host:  redis-headless.redis.svc.cluster.local
        port: 6379
      minio:
        enabled: true
        ingress:
          tls:
            enabled: true
            secretName: gitlab-minio-tls
        persistence:
          storageClass: nfs-bnvkube-client
          size: 100Gi
        credentials: {}
      kas:
        enabled: false
      registry:
        tls:
          enabled: true
          secretName: gitlab-registry-tls
        enabled: true
      email:
        from: "admin@bnvkube.lan"
    certmanager:
      install: false
    nginx-ingress: &nginx-ingress
      enabled: false
    prometheus:
      install: false
    redis:
      install: false
    postgresql:
      install: false
    gitlab-runner:
      install: false
    gitlab:
      toolbox:
        backups:
          cron:
            enabled: true
        replicas: 1
        antiAffinityLabels:
          matchLabels:
            app: gitaly
      gitaly:
        persistence:
          size: 30Gi
          storageClass: nfs-bnvkube-client
    ```

12. Устанавливаем
    ```bash
    helm upgrade --install gitlab gitlab/gitlab -f ./00-gitlab-values.yaml --namespace gitlab --create-namespace
    ```

---

### Удаление
   ```bash
   helm uninstall gitlab -n gitlab
   ```

---

[[Перейти в начало](../README.md)]