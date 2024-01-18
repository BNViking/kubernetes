[[Перейти в начало](../README.md)]

---

## Установка CNI [Calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/helm)

* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

После развертывание Kubernetes необходимо установить CNI (интерфейс контейнерных сетей)

1. Добавляем репозиторий
    ```bash
    helm repo add projectcalico https://docs.tigera.io/calico/charts
    ```

2. Обновляем репозитории
    ```bash
    helm repo update
    ```

3. Создаем namespace
    ```bash
    kubectl create namespace tigera-operator
    ``` 

5. Устанавливаем tigera-operator
    ```bash
    helm install calico projectcalico/tigera-operator --namespace tigera-operator
    ```

---

[[Перейти в начало](../README.md)]