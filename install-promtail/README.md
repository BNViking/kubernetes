[[Перейти в начало](../README.md)]

---

## Установка [Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/)

* [Документация](https://grafana.com/docs/loki/latest/send-data/promtail/installation/)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))
* [Artifact Hub](https://artifacthub.io/packages/helm/grafana/promtail)

Promtail - это агент, который отправляет содержимое локальных журналов в Grafana Loki или Grafana Cloud. \
Как правило, Promtail устанавливается на каждом узле, где необходимо собирать логи приложений или системные логи.\
В настоящее время Promtail может собирать логи из двух источников: локальных лог-файлов и systemd (только на машинах AMD64).

---

1. Добавляем репозиторий
    ```bash
    helm repo add grafana https://grafana.github.io/helm-charts
    ```

2. Обновляем репозитории
    ```bash
    helm repo update
    ```

3. Создаем файл для конфигурации [./00-promtail-values.yaml](./00-promtail-values.yaml)
    ```bash
    helm show values grafana/promtail > ./00-promtail-values.yaml
    ``` 
4. Редактируем файл [./00-promtail-values.yaml](./00-promtail-values.yaml)
   ```yaml
   daemonset:
      enabled: true
   configmap:
      enabled: true
   resources:
      limits:
         cpu: 200m
         memory: 128Mi
      requests:
         cpu: 100m
         memory: 128Mi
   config:
      clients:
        - url: http://loki-gateway/loki/api/v1/push
   ```

5. Устанавливаем
    ```bash
    helm install promtail grafana/promtail -f ./00-promtail-values.yaml --namespace logging
    ```

---

#### Удаление

```bash
helm uninstall promtail --namespace logging
```

---

[[Перейти в начало](../README.md)]