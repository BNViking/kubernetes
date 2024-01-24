[[Перейти в начало](../../README.md)]

---

## Проверка работоспособности DNS

* [Ссылка на тестирование DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

1. Устанавливаем манифест [dnsutils.yaml](https://k8s.io/examples/admin/dns/dnsutils.yaml)
    ```bash
    kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
    ```

2. Смотрим что dnsutils готов к работе
    ```bash
    kubectl get pods dnsutils -o wide
    ```
   
    ```
    NAME       READY   STATUS    RESTARTS   AGE   IP             NODE            NOMINATED NODE   READINESS GATES
    dnsutils   1/1     Running   0          81s   10.20.24.132   worker-03.lan   <none>           <none>
    ```

3. Проверяем путь до нашего сервиса kubernetes
    ```bash
    kubectl exec -i -t dnsutils -- nslookup kubernetes.default
    ```
    ```
    Server:         169.254.20.10
    Address:        169.254.20.10#53
    
    Name:   kubernetes.default.svc.cluster.local
    Address: 10.10.0.1
    ```
   
   Мы получили IP, а так же полное DNS имя, значит DNS настроен правильно.

---

Удаление dnsutils
```bash
kubectl delete -f https://k8s.io/examples/admin/dns/dnsutils.yaml
```

---

[[Перейти в начало](../../README.md)]