[[Перейти в начало](../README.md)]

---

## Установка Minio

* [Официальный сайт](https://min.io/)
* [ArtifactHub](https://artifacthub.io/packages/helm/bitnami/minio)
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
   kubectl create namespace minio
   ```

4. Создадим секрет для авторизации в minio
   ```bash
   kubectl create secret generic minio-admin-auth -n minio --from-literal=root-user=bnviking --from-literal=root-password=adminpassword
   ```

5. Создаем файл [./00-minio-values.yaml](./00-minio-values.yaml) с настройками
   ```bash
   helm show values bitnami/minio > ./00-minio-values.yaml
   ```

6. Редактируем настройки [./00-minio-values.yaml](./00-minio-values.yaml)
   ```yaml
   global:
     storageClass: "nfs-bnvkube-client"
   clusterDomain: cluster.local
   mode: distributed
   auth:
     existingSecret: "minio-admin-auth"
   statefulset:
     replicaCount: 2
   ingress:
     enabled: true
     ingressClassName: "nginx"
     hostname: minio.bnvkube.lan
     annotations:
       cert-manager.io/cluster-issuer: self-signed
     extraTls:
       - hosts:
         - minio.bnvkube.lan
         secretName: minio-tls
   persistence:
     enabled: true
     storageClass: "nfs-bnvkube-client"
     mountPath: /minio/data
     size: 50Gi
   volumePermissions:
     enabled: true
     containerSecurityContext:
       runAsUser: 0
   metrics:
     prometheusAuthType: public
     serviceMonitor:
       enabled: true
   ```

7. Устанавливаем
   ```bash
   helm install minio bitnami/minio -f ./00-minio-values.yaml --namespace minio --create-namespace
   ```
   
---

### Удаление
   ```bash
   helm uninstall minio -n minio
   ```

Смотрим какие остались тома
   ```bash
   kubectl get pvc -n minio
   ```

---

[[Перейти в начало](../README.md)]