[[Перейти в начало](../../README.md)]

---

## Установка Argo CD

* [Git Hub](https://github.com/argoproj/argo-cd?tab=readme-ov-file)
* [Artifact Hub](https://artifacthub.io/packages/helm/argo/argo-cd)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

### Пошаговая установка

1. Добавьте репозиторий и выберите версию для установки
   ```bash
   helm repo add argo https://argoproj.github.io/argo-helm
   ```

2. Обновляем кеш приложений (charts)
   ```bash
   helm repo update
   ```

3. Создадим namespace
   ```bash
   kubectl create namespace argo-cd
   ```

4. Создадим секретный ключ для подключения к **Redis**
   ```bash
   kubectl create secret generic argo-cd-redis-password -n argo-cd --from-literal=redis-password=redispassword
   ```

5. Создаем файл [./00-argo-cd-values.yaml](./00-argo-cd-values.yaml) с настройками
   ```bash
   helm show values argo/argo-cd > ./00-argo-cd-values.yaml
   ```

6. Редактируем настройки [./00-argo-cd-values.yaml](./00-argo-cd-values.yaml)
   ```yaml
   global:
     logging:
       format: json
   configs:
     params:
       server.insecure: true
   dex:
     enabled: false
   redis:
     enabled: false
   externalRedis:
     host: "redis-headless.redis.svc.cluster.local"
     existingSecret: 
   server:
     ingress:
       enabled: true
       annotations:
         cert-manager.io/cluster-issuer: self-signed
       ingressClassName: "nginx"
       hosts:
         - argocd.bnvkube.lan
       tls:
         - secretName: argo-cd-tls
           hosts:
             - argocd.bnvkube.lan
       https: true
   ```

7. Устанавливаем
   ```bash
   helm install argo-cd argo/argo-cd -f ./00-argo-cd-values.yaml --namespace argo-cd --create-namespace
   ```

---

### Удаление
   ```bash
   helm uninstall argo-cd -n argo-cd
   ```

   ```bash
   kubectl delete crd applications.argoproj.io applicationsets.argoproj.io appprojects.argoproj.io
   ```

---

[[Перейти в начало](../../README.md)]