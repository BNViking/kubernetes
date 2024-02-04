[[Перейти в начало](../../README.md)]

---

## Установка [Loki](https://grafana.com/oss/loki/)

* [Документация](https://grafana.com/docs/loki/latest/?pg=oss-loki&plcmt=quick-links)
* [Описание всех параметров конфигурации](https://grafana.com/docs/loki/latest/configure/)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))
* [Artifact Hub](https://artifacthub.io/packages/helm/grafana/loki)

Grafana Loki — это набор компонентов, которые могут быть скомпонованы в полнофункциональный стек логирования.\
В отличие от других систем ведения журналов, Loki построена на идее индексирования только метаданных о ваших журналах: 
меток (так же, как метки Prometheus). Затем сами данные журнала сжимаются и хранятся фрагментами в объектных хранилищах, 
таких как Amazon Simple Storage Service (S3) или Google Cloud Storage (GCS), или даже локально в файловой системе.\
Небольшой индекс и сильно сжатые чанки упрощают работу и значительно снижают стоимость Loki.

---

1. Добавляем репозиторий
    ```bash
    helm repo add grafana https://grafana.github.io/helm-charts
    ```

2. Обновляем кеш приложений (charts)
    ```bash
    helm repo update
    ```

3. Создаем файл [./00-loki-values.yaml](./00-loki-values.yaml) с настройками
    ```bash
    helm show values grafana/loki > 00-loki-values.yaml
    ``` 
4. Редактируем файл [./00-loki-values.yaml](./00-loki-values.yaml)
   ```yaml
   global:
     clusterDomain: "cluster.local"
     dnsService: "kube-dns"
     dnsNamespace: "kube-system"
   loki:
     auth_enabled: false
     server:
       grpc_server_max_recv_msg_size: 8388608
       grpc_server_max_send_msg_size: 8388608
     revisionHistoryLimit: 2
     limits_config:
       reject_old_samples_max_age: 120h #Хранение логов
     commonConfig:
       replication_factor: 2
     storage:
       type: filesystem #тип хранилища для логов
       filesystem:
         chunks_directory: /var/loki/chunks
         rules_directory: /var/loki/rules
   test:
     enabled: false
   monitoring:
     lokiCanary:
       enabled: false
   write:
     replicas: 2
     persistence:
       size: 5Gi
       storageClass: nfs-bnvkube-client
   read:
     replicas: 2
     persistence:
       size: 5Gi
       storageClass: nfs-bnvkube-client
   backend:
     replicas: 2
     persistence:
       size: 5G
       storageClass: nfs-bnvkube-client 
   singleBinary:
     replicas: 1
     persistence:
       size: 5G
       storageClass: nfs-bnvkube-client
   gateway:
     replicas: 2
   memberlist:
     service:
       publishNotReadyAddresses: false
   minio:
     enabled: false
   ```

5. Устанавливаем
    ```bash
    helm install loki grafana/loki -f 00-loki-values.yaml --namespace logging --create-namespace
    ```

После установки, надо сделать 2 реплики для StateFullSet Loki\
*Исправление ошибки ошибки:*\
*Error: INSTALLATION FAILED: execution error at (loki/templates/validate.yaml:22:4): Cannot run more than 1 Single Binary replica without an object storage backend.*

---

#### Удаление

```bash
helm uninstall loki --namespace logging
```

После удаления обычно остаются DaemonSet, удаляем их вручную, принудительно.
```bash
kubectl get pods -n logging
```
```
NAME              READY   STATUS        RESTARTS      AGE
loki-logs-45g7f   2/2     Terminating   2 (40m ago)   13h
loki-logs-mh4t4   2/2     Terminating   2 (40m ago)   12h
loki-logs-rnjpm   2/2     Terminating   2 (40m ago)   13h
```

Удаляем каждый командой
```bash
kubectl delete pod -n logging --force loki-logs-45g7f
```

---

[[Перейти в начало](../../README.md)]