[[Перейти в начало](../../README.md)]

---

## Установка GitLab

* [GitLab Chart](https://gitlab.com/gitlab-org/charts/gitlab)
* [GitLab документация](https://docs.gitlab.com/charts/)
* [Описание параметров](https://docs.gitlab.com/charts/charts/)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

---

Устанавливаем
* [NFS](../08-nfs/README.md)
* [Cert Manager](../install-cert-namager/README.md) + [Настройка](../09-cert-manager)
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
    registry:
      enabled: true
      ingress:
        annotations:
          nginx.ingress.kubernetes.io/ssl-redirect: "true"
          cert-manager.io/cluster-issuer: self-signed
          kubernetes.io/tls-acme: "true"
        tls:
          secretName: gitlab-registry-tls
    minio:
      persistence:
        storageClass: nfs-bnvkube-client
        size: 30Gi
      ingress:
        annotations:
          nginx.ingress.kubernetes.io/ssl-redirect: "true"
          cert-manager.io/cluster-issuer: self-signed
          kubernetes.io/tls-acme: "true"
        tls:
          secretName: gitlab-minio-tls
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
      webservice:
        enabled: true
        ingress:
          annotations:
            nginx.ingress.kubernetes.io/ssl-redirect: "true"
            cert-manager.io/cluster-issuer: self-signed
            kubernetes.io/tls-acme: "true"
          tls:
            secretName: gitlab-webservice-tls
      gitaly:
        persistence:
          size: 30Gi
          storageClass: nfs-bnvkube-client
      gitlab-shell:
        enabled: true
    ```

12. Устанавливаем
    ```bash
    helm upgrade --install gitlab gitlab/gitlab -f ./00-gitlab-values.yaml --namespace gitlab --create-namespace
    ```
---
### Использование gitlab registry без проверки сертификата

* [Инструкция настройки реестра образов](https://github.com/containerd/cri/blob/master/docs/registry.md#configure-registry-tls-communication)

Для отключения проверки сертификата необходимо добавить настройки containerd на всех узлах.

1. Создаем папку
   ```bash
   mkdir -p /etc/containerd/certs.d/registry.webc.int
   ```
   
2. Создаем файл `/etc/containerd/certs.d/registry.webc.int/hosts.toml`
   ```yaml
   server = "https://registry.webc.int"
   
   [host."https://registry.webc.int"]
     skip_verify = true
   ```

3. Редактируем файл настроек `/etc/containerd/config.toml`
   ```yaml
   [plugins."io.containerd.grpc.v1.cri".registry]
     config_path = "/etc/containerd/certs.d"
   ```

4. Перезапускаем containerd
   ```bash
   service containerd restart
   ```

### Настройка с предустановленным **minio**

1. Создать секрет авторизации в minio для раздела **global.appConfig.{lfs,artifacts,uploads,packages}.connection**
    ```bash
    cat << EOF | kubectl apply -f - 
    apiVersion: v1
    kind: Secret
    metadata:
      name: appconfig-minio-credentials
      namespace: gitlab
      labels:
        manual: "yes"
    type: Opaque
    stringData:
      connection: |
        provider: AWS
        region: ru-nsk-1 #Обязательно укажите свой регион
        endpoint: http://minio-headless.minio:9000
        path_style: true
        aws_access_key_id: "ACCESS_KEY" 
        aws_secret_access_key: "SECRET_KEY"
    EOF
    ```

2. Создать секрет авторизации в minio для раздела **gitlab.toolbox.backups.objectStorage.config**
    ```bash
    cat << EOF | kubectl apply -f -
    apiVersion: v1
    kind: Secret
    metadata:
      name: toolbox-minio-credentials
      namespace: gitlab
      labels:
        manual: "yes"
    type: Opaque
    stringData:
      config: |
        s3:
          bucket: gitlab-registry
          accesskey: "ACCESS_KEY"
          secretkey: "SECRET_KEY"
          region: ru-nsk-1 #Обязательно укажите свой регион
          regionendpoint: "http://minio-headless.minio:9000"
          v4auth: true
    EOF
    ```

3. Создаем **buckets** в minio
    ```
    gitlab-artifacts
    gitlab-backups
    gitlab-lfs
    gitlab-packages
    gitlab-registry
    gitlab-tmp
    gitlab-uploads
    ```

4. Редактируем настройки [./00-gitlab-values.yaml](./00-gitlab-values.yaml)
    ```yaml
    global:
      minio:
        enabled: false
      appConfig:
        lfs:
          enabled: true
          proxy_download: true
          bucket: gitlab-lfs
          connection:
            secret: appconfig-minio-credentials
            key: connection
        artifacts:
          enabled: true
          proxy_download: true
          bucket: gitlab-artifacts
          connection:
            secret: appconfig-minio-credentials
            key: connection
        uploads:
          enabled: true
          proxy_download: true
          bucket: gitlab-uploads
          connection:
            secret: appconfig-minio-credentials
            key: connection
        packages:
          enabled: true
          proxy_download: true
          bucket: gitlab-packages
          connection:
            secret: appconfig-minio-credentials
            key: connection
      registry:
        bucket: gitlab-registryz
    gitlab:
      toolbox:
        backups:
          objectStorage:
            config:
              secret: toolbox-minio-credentials
              key: config
    ```

5. Устанавливаем
   ```bash
   helm upgrade --install gitlab gitlab/gitlab -f ./00-gitlab-values.yaml --namespace gitlab --create-namespace
   ```

6. После установки загрузка в **registry** не будет работать, исправляем конфигурацию **ConfigMaps.gitlab-registry**, меняем блок **data.storage**, заменяем **filesystem** на **s3**
    ```yaml
    data:
      storage:
        ...
        s3:
          accesskey: "ACCESS_KEY"
          secretkey: "SECRET_KEY"
          region: ru-nsk-1
          regionendpoint: http://minio-headless.minio:9000
          bucket: gitlab-registry
          secure: true
          v4auth: true
          rootdirectory: /
    ```

7. Перезапускаем DaemonSet gitlab-registry

---

Возможно для клонирования проекта в **windows** потребуется настроить **git**
```bash
 git config --global http.sslbackend schannel
```

---

### Удаление
   ```bash
   helm uninstall gitlab -n gitlab
   ```

---

[[Перейти в начало](../../README.md)]