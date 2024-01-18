[[Перейти в начало](../README.md)]

---

## Установка [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) NGINX Controller

* [Официальный сайт](https://kubernetes.github.io/ingress-nginx/deploy/)
* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))

1. Добавляем репозиторий
    ```bash
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    ```

2. Обновляем список приложений
    ```bash
    helm repo update
    ```

3. Смотрим список доступных приложений ingress
    ```bash
    helm search repo | grep ingress
    ```

4. Создаем файл с настройками
    ```bash
    helm show values ingress-nginx/ingress-nginx > ./00-ingress-nginx-values.yaml
    ```

5. Редактируем [./00-ingress-nginx-values.yaml](./00-ingress-nginx-values.yaml)
   ```yaml
   controller:
       ingressClass: nginx #можно поменять на свой
       metrics:
           enabled: true #включение метрик
       admissionWebhooks:
           enabled: false #отключение проверок kind: ingress 
   ```
6. Устанавливаем
    ```bash
    helm install ingress-nginx ingress-nginx/ingress-nginx -f ./00-ingress-nginx-values.yaml -n ingress-nginx --create-namespace
    ```

7. Проверяем список сервисов
    ```bash
    kubectl -n ingress-nginx get svc
    ```
    ```
    NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
    ingress-nginx-controller           LoadBalancer   10.10.122.124   192.168.1.180   80:30980/TCP,443:31686/TCP   3m14s
    ingress-nginx-controller-metrics   ClusterIP      10.10.115.147   <none>          10254/TCP                    3m14s
    ```
---
*Если в систему установлен **mtallb**, проверяем выделенный контроллеру IP адрес.
Проверяем ответ контроллера.*

```bash
curl http://192.168.1.180
```
```html
<html>
   <head><title>404 Not Found</title></head>
   <body>
      <center><h1>404 Not Found</h1></center>
      <hr><center>nginx</center>
   </body>
</html>
```
Ответ получаем, но приложений пока что для обработки запроса нет, поэтому 404 =)

---

#### Пример использования
```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
```

---

### Удаление

```bash
helm uninstall ingress-nginx -n ingress-nginx
```

---

[[Перейти в начало](../README.md)]