[[Перейти в начало](../README.md)]

---

## Установка MetalLB

* [Инструкция на официальном сайте](https://metallb.org/installation/)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

Перед установкой необходимо ключить режим IPVS и поддержку ARP

*Если вы используете kube-proxy в режиме IPVS, начиная с Kubernetes v1.14.2, вы должны включить строгий режим ARP.\
Обратите внимание, что вам это не нужно, если вы используете kube-router в качестве service-proxy, потому что он включает строгий ARP по умолчанию.*

```bash
kubectl edit configmap -n kube-system kube-proxy
```
```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))
---
### Пошаговая установка

1. Добавляем репозиторий
    ```bash
    helm repo add metallb https://metallb.github.io/metallb
    ```

2. Обновляем список пакетов
    ```bash
    helm repo update
    ```

3. Создаем файл с настройками
    ```bash
    helm show values metallb/metallb > ./00-metallb-values.yaml
    ```

4. Редактируем [00-metallb-values.yaml](./00-metallb-values.yaml)

5. Устанавливаем
    ```bash
    helm install metallb metallb/metallb -f ./00-metallb-values.yaml --namespace metallb --create-namespace
    ```

---

#### Удаление

```bash
helm uninstall metallb -n metallb
```

---

[[Перейти в начало](../README.md)]