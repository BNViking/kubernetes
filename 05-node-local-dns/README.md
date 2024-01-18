[[Перейти в начало](../README.md)]

---

## Настройка Node Local Dns

* [Node Local DNS на kubernetes.io](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/)
* [Как настроить node-local-dns](https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/dns/nodelocaldns/README.md)

В официальном репозитории есть шаблон [примера манифеста для node-local-dns](https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/dns/nodelocaldns/nodelocaldns.yaml)\
В нем надо заменить значения:

* `__PILLAR__LOCAL__DNS__`
* `__PILLAR__DNS__DOMAIN__`
* `__PILLAR__DNS__SERVER__`
* `__PILLAR__CLUSTER__DNS__`
* `__PILLAR__UPSTREAM__SERVERS__`

*Они имеют разные значения, в зависимости от режима (IPVS/IPTABLE) работы kube-proxy*\

---

В настройках [kube-init.yaml](../03-first-control-plane/README.md) **kube-proxy** указан режим **IPVS**.\
В этом режиме `node-local-dns` модули прослушивают только файлы `<node-local-address>`.\
Интерфейс `node-local-dns` не может привязать IP-адрес кластера **kube-dns**, поскольку интерфейс, используемый для балансировки нагрузки **IPVS**, уже использует этот адрес.\
`__PILLAR__UPSTREAM__SERVERS__` будет заполнен модулями `node-local-dns`.

---

### Пошаговая инструкция для режима IPVS
1. Скачиваем шаблон
   ```bash
   wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns/nodelocaldns/nodelocaldns.yaml
   ```
   
2. Копируем шаблон ([nodelocaldns.yaml](./nodelocaldns.yaml)) в ./01-config-node-local-dns.yaml
    ```bash
    cp ./nodelocaldns.yaml ./01-config-node-local-dns.yaml
    ```

3. Редактируем файл 01-config-node-local-dns.yaml

   1. `__PILLAR__LOCAL__DNS__` - Указывали в настройках [kube-init.yaml](../03-first-control-plane/README.md) (KubeletConfiguration)

       ```yaml
       clusterDNS:
           - 169.254.20.10 #IP DNS Сервера
       ```

   2. `__PILLAR__DNS__DOMAIN__` - Указывали в настройках [kube-init.yaml](../03-first-control-plane/README.md) (KubeletConfiguration)

       ```yaml
       clusterDomain: cluster.local
       ```

   3. `__PILLAR__DNS__SERVER__` - меняем на пустоту (Просто удаляем `__PILLAR__DNS__SERVER__`)\
      *(В настройках, в строке 146 (`args: [-localip] ...`) надо убрать запятую)*
 
   4. `__PILLAR__CLUSTER__DNS__` 

       ```bash
       kubectl get svc -n kube-system kube-dns -o jsonpath='{.spec.clusterIP}'
       ```
       ```
       Результат: 10.10.0.10
       ```
   
   5. `__PILLAR__UPSTREAM__SERVERS__`
       ```
       /etc/resolv.conf
       ```       
4. Заменяем порты для health 8080 на 9254, так как в DaemonSet параметр `spec.hostNetwork: true`, это значит что приложение будет использовать порты узла.\
   *(На мой взгляд 8080 не должен быть использован в качестве health)*

5. После всех изменений применяем файл [01-config-node-local-dns.yaml](./01-config-node-local-dns.yaml)
    ```bash
    kubectl apply -f ./01-config-node-local-dns.yaml
    ```

*В DaemonSet параметр `spec.hostNetwork: true`, это значит что приложение будет использовать порт узла,
так же используется порт 8080 для health*

---

После включения `node-local-dns` поды будут работать в kube-system пространстве имен на каждом из узлов кластера.\
Этот модуль запускает CoreDNS в режиме кэша, поэтому все метрики CoreDNS, предоставляемые различными плагинами, будут доступны для каждого узла.

---
Удаление, если что-то пошло не так =)

```bash
kubectl delete -f ./01-config-node-local-dns.yaml
```

---

Проверить открытие порта
```bash
ss -nltp | grep :53
```
```
LISTEN 0      4096      10.10.0.10:53         0.0.0.0:*
LISTEN 0      4096   169.254.20.10:53         0.0.0.0:*
LISTEN 0      4096   127.0.0.53%lo:53         0.0.0.0:*
```

---

[[Перейти в начало](../README.md)]