[[Перейти в начало](../README.md)]

---

## Установка Prometheus Stack

* Устанавливать будем средствами Helm ([Ссылка на установку](../install-helm/README.md))
* [ArtifactHub](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)

1. Создадим namespace monitoring
   ```bash
   kubectl create namespace monitoring
   ```

2. Создадим секрет для grafana, в котором будет храниться логин и пароль для администратора
   ```bash
   kubectl create secret generic grafana-user-admin -n monitoring --from-literal=username=bnviking --from-literal=password=pwdpwdpwd
   ```

3. Добавляем репозиторий
    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    ```

4. Обновляем кеш приложений (charts)
    ```bash
    helm repo update
    ```

5. Создаем файл [./00-prometheus-stack-values.yaml](./00-prometheus-stack-values.yaml) c настройками
    ```bash
    helm show values prometheus-community/kube-prometheus-stack > ./00-prometheus-stack-values.yaml
    ```

6. Редактируем файл [./00-prometheus-stack-values.yaml](./00-prometheus-stack-values.yaml)
    ```yaml
    fullnameOverride: "prometheus" #Избавляемся от длинных стандартных имен
    alertmanager:
        fullnameOverride: "alertmanager" #Избавляемся от длинных стандартных имен
        ingress:
            enable: true #если не надо, то можно выключить
            ingressClassName: nginx
            annotations:
                cert-manager.io/cluster-issuer: self-signed #включаем генерацию сертификата
                kubernetes.io/tls-acme: "true"
            hosts:
                - alertmanager.bnvkube.lan
            tls:
                - secretName: alert-manager-tls
                  hosts:
                    - alertmanager.bnvkube.lan
        alertmanagerSpec:
                storage:
                    volumeClaimTemplate:
                        spec:
                            storageClassName: nfs-bnvkube-client 
                            accessModes: ["ReadWriteOnce"]
                            resources:
                                requests:
                                    storage: 5Gi
                        selector: {}
    grafana:
        fullnameOverride: "grafana" #Избавляемся от длинных стандартных имен
            admin:
                existingSecret: grafana-user-admin #имя нашего созданного секрета
                userKey: username #наименование поля откуда брать логин
                passwordKey: password #наименование поля откуда брать пароль
        ingress:
            enable: true #если не надо, то можно выключить
            ingressClassName: nginx
            annotations:
                cert-manager.io/cluster-issuer: self-signed #включаем генерацию сертификата
                kubernetes.io/tls-acme: "true"
            hosts:
                - grafana.bnvkube.lan
            tls:
                - secretName: grafana-tls
                  hosts:
                    - grafana.bnvkube.lan
    kubeControllerManager:
        endpoints:
            - 192.168.1.12
            - 192.168.1.13
            - 192.168.1.14
    kubeEtcd:
        endpoints:
            - 192.168.1.12
            - 192.168.1.13
            - 192.168.1.14
    kubeScheduler:
        endpoints:
            - 192.168.1.12
            - 192.168.1.13
            - 192.168.1.14
    kubeProxy:
        endpoints:
            - 192.168.1.12
            - 192.168.1.13
            - 192.168.1.14
    kube-state-metrics:
        fullnameOverride: "kube-state-metrics"
        prometheus:
            monitor:
                relabelings:
                  - action: replace
                    regex: (.*)
                    replacement: $1
                    sourceLabels:
                      - __meta_kubernetes_pod_node_name
                    targetLabel: kubernetes_node
        selfMonitor:
            enabled: true
    nodeExporter:
        serviceMonitor:
            relabelings:
                - action: replace
                  regex: (.*)
                  replacement: $1
                  sourceLabels:
                    - __meta_kubernetes_pod_node_name
                  targetLabel: kubernetes_node
    prometheus-node-exporter:
        fullnameOverride: "prometheus-node-exporter"
        prometheus:
            monitor:
                relabelings:
                  - action: replace
                    regex: (.*)
                    replacement: $1
                    sourceLabels:
                      - __meta_kubernetes_pod_node_name
                    targetLabel: kubernetes_node
    prometheusOperator:
        fullnameOverride: "prometheus-operator"
    resources:
        limits:
            cpu: 200m
            memory: 200Mi
        requests:
            cpu: 100m
            memory: 100Mi
    prometheusConfigReloader:
        resources:
            requests:
                cpu: 200m
                memory: 50Mi
            limits:
                cpu: 200m
                memory: 50Mi
    prometheus:
        ingress:
            enable: true #если не надо, то можно выключить
            ingressClassName: nginx
            annotations:
                cert-manager.io/cluster-issuer: self-signed #включаем генерацию сертификата
                kubernetes.io/tls-acme: "true"
            hosts:
                - prometheus.bnvkube.lan
            tls:
                - secretName: prometheus-tls
                  hosts:
                    - prometheus.bnvkube.lan
        prometheusSpec:
            enableAdminAPI: true
            serviceMonitorSelectorNilUsesHelmValues: false
            podMonitorSelectorNilUsesHelmValues: false
            probeSelectorNilUsesHelmValues: false
            retention: 5d
            storageSpec:
                volumeClaimTemplate:
                    spec:
                        storageClassName: nfs-bnvkube-client 
                        accessModes: ["ReadWriteOnce"]
                        resources:
                            requests:
                                storage: 5Gi
                    selector: {}
    ```

7. Устанавливаем
    ```bash
    helm install prometheus prometheus-community/kube-prometheus-stack -f ./00-prometheus-stack-values.yaml --namespace monitoring --create-namespace  
    ```

---

Смотрим что бы все поднялось
```bash
kubectl --namespace monitoring get pods -o wide
```
```
NAME                                                     READY   STATUS    RESTARTS   AGE   IP             NODE             NOMINATED NODE   READINESS GATES
alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   0          50s   10.20.37.206   worker-02.lan    <none>           <none>
prometheus-grafana-77c588fccf-zs8np                      3/3     Running   0          65s   10.20.24.146   worker-03.lan    <none>           <none>
prometheus-kube-prometheus-operator-86cbd94f79-xb7fz     1/1     Running   0          65s   10.20.24.147   worker-03.lan    <none>           <none>
prometheus-kube-state-metrics-6db866c85b-szzkj           1/1     Running   0          65s   10.20.24.145   worker-03.lan    <none>           <none>
prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0          50s   10.20.37.207   worker-02.lan    <none>           <none>
prometheus-prometheus-node-exporter-6bsns                1/1     Running   0          65s   192.168.1.13   control-02.lan   <none>           <none>
prometheus-prometheus-node-exporter-7sfjp                1/1     Running   0          65s   192.168.1.17   worker-03.lan    <none>           <none>
prometheus-prometheus-node-exporter-dc9k9                1/1     Running   0          65s   192.168.1.14   control-03.lan   <none>           <none>
prometheus-prometheus-node-exporter-h9zbk                1/1     Running   0          65s   192.168.1.15   worker-01.lan    <none>           <none>
prometheus-prometheus-node-exporter-hsghv                1/1     Running   0          65s   192.168.1.16   worker-02.lan    <none>           <none>
prometheus-prometheus-node-exporter-l295k                1/1     Running   0          65s   192.168.1.12   control-01.lan   <none>           <none>
```

Созданные сертификаты
```bash
kubectl get certificate -n monitoring
```
```
NAME                READY   SECRET              AGE
alert-manager-tls   True    alert-manager-tls   102s
grafana-tls         True    grafana-tls         102s
prometheus-tls      True    prometheus-tls      102s
```

Созданные разделы 
```bash
kubectl get pv -n monitoring
```
```
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                                                       STORAGECLASS         VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-b876bad4-021e-4c50-9d3e-966bb7c34958   5Gi        RWO            Delete           Bound    monitoring/prometheus-prometheus-prometheus-db-prometheus-prometheus-prometheus-0           nfs-bnvkube-client   <unset>                          3m12s
pvc-dd9ec905-13d1-4422-9f32-5c7c3461aa4a   5Gi        RWO            Delete           Bound    monitoring/alertmanager-prometheus-alertmanager-db-alertmanager-prometheus-alertmanager-0   nfs-bnvkube-client   <unset>                          3m13s
```

---

Входим на сервисы и проверяем работоспособность

* [prometheus.bnvkube.lan](https://prometheus.bnvkube.lan)
* [alertmanager.bnvkube.lan](https://alertmanager.bnvkube.lan)
* [grafana.bnvkube.lan](https://grafana.bnvkube.lan) (username: `bnviking` password: `pwdpwdpwd`)

---

### Удаление
```bash
helm uninstall prometheus -n monitoring
```

---

[[Перейти в начало](../README.md)]