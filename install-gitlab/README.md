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
    helm show values gitlab/gitlab > ./00-gitlab-values.yaml
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
            secretName: gitlab-minio-tls #не работает, надо после установки поправить
        persistence:
          storageClass: nfs-bnvkube-client #не работает, надо после установки поправить
          size: 20Gi #не работает, надо после установки поправить
        credentials: {}
      kas:
        enabled: false
      registry:
        tls:
          enabled: true
          secretName: gitlab-registry-tls #не работает, надо после установки поправить
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

#### Исправление конфигурации 

_Через настройки не применились значения, поэтому правим после установки_

1. **PersistentVolumeClaim gitlab-minio**, добавить **storageClassName** и изменить размер тома
   ```yaml
    spec:
      storageClassName: nfs-bnvkube-client
      resources:
        requests:
          storage: 20Gi
    ```

2. **Ingress gitlab-minio**, заменить **secretName**
    ```yaml
      tls:
        - hosts:
            - minio.bnvkube.lan
          secretName: gitlab-minio-tls
    ```

3. **Ingress gitlab-registry**, заменить **secretName**
    ```yaml
      tls:
        - hosts:
            - registry.bnvkube.lan
          secretName: gitlab-registry-tls
    ```


---

### Удаление
   ```bash
   helm uninstall gitlab -n gitlab
   ```

---

[[Перейти в начало](../README.md)]