[[Перейти в начало](../../README.md)]

---

## Создание первого узла Control Plane (kubeadm)

* [Описание конфигурации на kubernetes.io](https://kubernetes.io/docs/reference/config-api/)
* [kubeadm Configuration на kubernetes.io](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/)
* [Описание компонентов на kubernetes.io](https://kubernetes.io/docs/reference/command-line-tools-reference/)
* [Kubelet Конфигурация](https://cluster-api.sigs.k8s.io/tasks/bootstrap/kubeadm-bootstrap/kubelet-config)
 
Для просмотра рекомендуемого конфигурационного файла можно использовать команду
```bash
kubeadm config print init-defaults
```

### Разворачивание первого узла с ролью Control Plane

1. Создаем файл с конфигурацией 
   ```bash
   touch ./kube-init.yaml
   ```
2. Пример файла [kube-init.yaml](./kube-init.yaml) с конфигурацией

   *Обратить внимание на следующие параметры*
   
   Настройка инициализации узла

   ```yaml
   kind: InitConfiguration
   bootstrapTokens:
       ttl: 24h0m0s #Время действия токена (понадобится для добавления остальных узлов)
   localAPIEndpoint:
       advertiseAddress: 192.168.1.12 #IP Узла
       bindPort: 6443
   nodeRegistration:
       criSocket: unix:///var/run/containerd/containerd.sock #Путь к сокету контейнера
       name: control-01.lan #Имя узла
   ```
   
   Тут в основном настройка сети   

   ```yaml
   kind: ClusterConfiguration
   clusterName: bnvkube.lan #Имя кластера
   kubernetesVersion: 1.29.0 #Версия kubernetes
   controlPlaneEndpoint: 192.168.1.12:7443 #IP на балансировщик (Обязательно если узлов Contr Plane будет больше 1)
   certificatesDir: /etc/kubernetes/pki #Указывает, где хранить или искать все необходимые сертификаты.
   apiServer:
       extraArgs:
           bind-address: 0.0.0.0 #Для работы метрик
           service-cluster-ip-range: "10.10.0.0/16"
           service-node-port-range: 30000-32767
   etcd:
       extraArgs:
           listen-metrics-urls: "http://0.0.0.0:2381" #Для работы метрик
   controllerManager:
       extraArgs:
           bind-address: 0.0.0.0 #Для работы метрик
   networking: #Настройка внутренней сети кластера
       dnsDomain: cluster.local
       podSubnet: 10.20.0.0/16
       serviceSubnet: 10.10.0.0/16
   scheduler:
       extraArgs:
           bind-address: 0.0.0.0 #Для работы метрик
   ```   

   Включаем режим ipvs и ARP   

   ```yaml
   kind: KubeProxyConfiguration
   bindAddress: 0.0.0.0
   metricsBindAddress: 0.0.0.0:10249 #Для работы метрик
   clusterCIDR: 10.20.0.0/16 #Диапазон CIDR модулей pod в кластере
   mode: ipvs
   ipvs:
       strictARP: True #Если вы используете kube-proxy в режиме IPVS, начиная с Kubernetes v1.14.2, вы должны включить строгий режим ARP.
   ```
   
   Устанавливаем clusterDNS, так как будем использовать [node-local-dns](../05-node-local-dns/README.md)

   ```yaml
   kind: KubeletConfiguration
   clusterDNS:
       - 169.254.20.10 #IP DNS Сервера
   clusterDomain: cluster.local
   ```
3. Запускаем команду инициализации kubernetes
   ```bash
   sudo kubeadm init --config ./kube-init.yaml
   ```

   *Если получили ошибку, необходимо запустить команду сброса, перед новой установкой узла*
   ```bash
   sudo kubeadm reset;
   ```
   ```bash
   sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X && sudo ipvsadm --clear
   ```

4. Копируем настройки для подключения kubectl
   ```bash
   mkdir -p $HOME/.kube;
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config;
   sudo chown $(id -u):$(id -g) $HOME/.kube/config;
   ```
5. После установки будет предложено подключить дополнительные узлы
   *Для Contrl Plane*
   ```bash
   sudo kubeadm join 192.168.1.12:7443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:e11c1247d93f672d0132dac847522870b77c7a30d19bff6a88fb09242837cc56 \
        --control-plane
   ```
   *Для остальных узлов*   
   ```bash
   sudo kubeadm join 192.168.1.12:7443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:e11c1247d93f672d0132dac847522870b77c7a30d19bff6a88fb09242837cc56
   ```
   
   *Время действия токена берется из параметра InitConfiguration bootstrapTokens.ttl: 24h0m0s*

Первый узел типа Control Plane развернут.

Kubernetes предложит [установить дополнения](https://kubernetes.io/docs/concepts/cluster-administration/addons/), но это мы сделаем попозже =)

Проверяем вывод информации о подключенных узла
   ```bash
   kubectl get nodes -o wide
   ```
Получаем ответ

```
NAME             STATUS     ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
control-01.lan   NotReady   control-plane   32s   v1.29.0   192.168.1.12   <none>        Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.2
```
Все хорошо, статус пока что NotReady, но когда подключим все узлы и настроим CNI, все будет ОК =)

---

[[Перейти в начало](../../README.md)]