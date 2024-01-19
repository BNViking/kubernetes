[[Перейти в начало](../README.md)]

---

## Установка Cert Manager

* [Helm install](https://cert-manager.io/docs/installation/helm/)
* [GitHub](https://github.com/cert-manager/cert-manager)
* [ArtifactHub](https://artifacthub.io/packages/helm/cert-manager/cert-manager)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

### Пошаговая установка

1. Добавьте репозиторий и выберите версию для установки
   ```bash
   helm repo add jetstack https://charts.jetstack.io
   ```

2. Обновляем кеш приложений (charts)
   ```bash
   helm repo update
   ```

3. Список доступных версий
   ```bash
   helm search repo jetstack -l
   ```

4. Устанавливаем
   ```bash
   helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.13.3 --set installCRDs=true
   ```

   *installCRDs - Если значение равно true, ресурсы CRD будут установлены как часть диаграммы Helm. Если этот параметр включен, при удалении CRD ресурсы будут удалены, в результате чего все установленные пользовательские ресурсы будут удалены*

---

Можно отдельно установить **CRD**, тогда при установке Helm параметр **installCRDs** надо указать **false**, но сначала установить **CRDs**.
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.crds.yaml
```

---

### Удаление cert-manager
   ```bash
   helm uninstall cert-manager -n cert-manager
   ```

### Удаление CRDs

1. Смотрим все созданные CRDs
   ```bash
   kubectl get crd | grep cert-manager
   ```
   
2. Удаляем CRDs
   ```bash
   kubectl delete crd certificaterequests.cert-manager.io;
   kubectl delete crd certificates.cert-manager.io;
   kubectl delete crd challenges.acme.cert-manager.io;
   kubectl delete crd clusterissuers.cert-manager.io;
   kubectl delete crd issuers.cert-manager.io;
   kubectl delete crd orders.acme.cert-manager.io;
   ```
---

[[Перейти в начало](../README.md)]