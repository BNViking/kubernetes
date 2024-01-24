[[Перейти в начало](../README.md)]

---

## Установка GitLab Runner

---

### Полезные ссылки

* [GitLab Chart](https://gitlab.com/gitlab-org/charts/gitlab)
* [GitLab документация](https://docs.gitlab.com/charts/)
* [Конфигурация GitLab Runner](https://docs.gitlab.com/runner/configuration/)
* [Установка GitLab Runner с помощью Helm](https://docs.gitlab.com/ee/ci/runners/new_creation_workflow.html#installing-gitlab-runner-with-helm-chart)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

---

### Установка зависимостей
* [GitLab](../install-gitlab/README.md)

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
   
4. В **GitLab** создаем новый **Runner** (Для проекта, группы, приложения или общий). Копируем его токен.
   ```
   gitlab-runner register  
     --url https://gitlab.bnvkube.lan  
     --token glrt-gH3VgrVhYz9y3MRboDXw
   ```
   
5. Создаем секрет
   ```bash
   cat << EOF | kubectl apply -f -
   apiVersion: v1
   kind: Secret
   metadata:
     name: web-gitlab-runner #Наименование
     namespace: gitlab
   type: Opaque
   stringData:
     # Токен при создании ранера
     runner-token: "glrt-gH3VgrVhYz9y3MRboDXw" 
     # нужно оставить пустой строкой из соображений совместимости
     runner-registration-token: ""
   
     # Авторизация в minio, хранится в секрете gitlab-minio-secret
     accesskey: "2GzNcrAlcXJwH1iFIyAXeP8YGqTKKLiHyxejvaexjB6R3RZ2QlsmZS91XdTUN9QD"
     secretkey: "BYklAqxkYXx4IhdbjUHZENMnZfE5DruhUtJKtcLyPk7SstFUvTs4AwjBqI78pFgq"
   EOF
   
   ```

6. Создаем файл [./00-gitlab-runner-values.yaml](./00-gitlab-runner-values.yaml) с настройками
   ```bash
   helm show values gitlab/gitlab-runner > ./00-gitlab-runner-values.yaml
   ```

7. Редактируем настройки [./00-gitlab-runner-values.yaml](./00-gitlab-runner-values.yaml)
   ```yaml
   revisionHistoryLimit: 3
   gitlabUrl: http://gitlab-webservice-default.gitlab.svc.cluster.local:8080
   concurrent: 5
   logLevel: info
   logFormat: json
   rbac:
     create: true
     rules:
      - resources: ["configmaps", "events", "pods", "pods/attach", "pods/exec", "secrets", "services"]
        verbs: ["get", "list", "watch", "create", "patch", "update", "delete"]
      - apiGroups: [""]
        resources: ["pods/exec", "pods/attach"]
        verbs: ["create", "patch", "delete"]
      - apiGroups: [""]
        resources: ["pods/log"]
        verbs: ["get"]
   metrics:
     enabled: true
   runners:
     config: |
       [[runners]]
         output_limit = 10000
         [runners.kubernetes]
           image = "ubuntu:22.04"
         [runners.cache]
           Type = "s3"
           Path = "gitlab-runner"
           Shared = true
           [runners.cache.s3]
             ServerAddress = "gitlab-minio-svc.gitlab.svc.bnvkube.local:9000"
             BucketName = "web-runner-cache"
             BucketLocation = "us-east-1"
             Insecure = true
             AuthenticationType = "access-key"
     executor: kubernetes
     secret: web-gitlab-runner
     cache:
       secretName: web-gitlab-runner
   ```

8. Устанавливаем
   ```bash
   helm upgrade --install web-gitlab-runner gitlab/gitlab-runner -f ./00-gitlab-runner-values.yaml --namespace gitlab --create-namespace
   ```
---

### Удаление
   ```bash
   helm uninstall web-gitlab-runner -n gitlab
   ```
   ```bash
   kubectl delete secret web-gitlab-runner -n gitlab
   ```
---

[[Перейти в начало](../README.md)]