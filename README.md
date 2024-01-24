# Тестовый стенд kebernetes

### Структура кластера
* Минимальные требования для узла 2 ядра и 4GB RAM и 30GB DISK
* В качестве OS на узлах будет использоваться [Ubuntu Server 22.04](https://ubuntu.com/download/server) версия ядра 5.15.0 
* В кластере будет 3 узла с ролью [Control Plane](https://kubernetes.io/docs/concepts/overview/components/#control-plane-components) и 3 узла с ролью Worker
* На одном из узлов будет установлен [haProxy](https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers) в роли балансировщика, он будет обслуживать наш кластер, слушать порт 7443
и перенаправлять трафик на 3 узла [Control Plane](https://kubernetes.io/docs/concepts/overview/components/#control-plane-components) на порт 6443
* На всех узлах будет свой [локальный DNS](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/)
* Управлять сетью контейнеров ([CNI](https://github.com/containernetworking/cni?tab=readme-ov-file)) будет [Calico](https://www.tigera.io/tigera-products/calico/)
* Будет использоваться [NFS (Network File System)](https://ubuntu.com/server/docs/service-nfs) для создания [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
* В роли [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) контроллера будет выступать [Nginx](https://kubernetes.github.io/ingress-nginx/)
* Выдавать внешние IP адреса для сервисов типа [LoadBalancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/) будет [MetalLB](https://metallb.org/)
* Управление [сертификатами](https://kubernetes.io/docs/tasks/administer-cluster/certificates/) будет через [cert-manager](https://cert-manager.io/docs/) (Создадим самоподписывающий сертификат)
* Система мониторинга - [Prometheus Stack](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)
* Система логирования - [Loki](https://grafana.com/docs/loki/latest/) + [Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/)

![cluster.png](./doc/img/cluster.png)

### Список узлов:

```yaml
   192.168.1.12:   control-01.lan;
   192.168.1.13:   control-02.lan;
   192.168.1.14:   control-03.lan;
   192.168.1.15:   worker-01.lan;
   192.168.1.16:   worker-02.lan;
   192.168.1.17:   worker-03.lan;
```

*У вас должны быть настроены DNS записи, что бы обращаться к узлу по имени хоста (hostname: control-01.lan)*

### Развертывание и настройка [kubernetes (v1.29.0)](https://kubernetes.io/blog/2023/12/13/kubernetes-v1-29-release/)

* [Подготовка узла](./doc/00-preparing-machine/README.md)
* [Создание виртуальных машин](./doc/01-create-vm-machine/README.md)
* [Настройка haProxy в роле балансировщика](./doc/02-haProxy/README.md)
* [Создание первого узла Control Plane (kubeadm)](./doc/03-first-control-plane/README.md)
* [Добавление узлов в Kubernetes (kubeadm)](./doc/04-add-node/README.md)
* [Настройка Node Local Dns](./doc/05-node-local-dns/README.md)
* [Установка CNI](./doc/06-calico/README.md)
* [Проверка DNS](./doc/07-check-dns/README.md)
* [Настройка NFS](./doc/08-nfs/README.md)
* [Настройка Cert Manager](./doc/09-cert-manager/README.md)
* [Настройка MetalLB](./doc/10-metal-lb/README.md)
* [Установка Ingress Controller (nginx)](./doc/install-ingress-nginx-controller/README.md)
* [Мониторинг ресурсов](./doc/11-monitoring/README.md)
* [Логирование](./doc/12-logi/README.md)

---

### Установка приложений

* [Helm](./doc/install-helm/README.md)
* [Calico](./doc/06-calico/README.md)
* [Cert Manager](./doc/install-cert-namager/README.md)
* [MetalLB](./doc/install-metal-lb/README.md)
* [Ingress NGINX Controller](./doc/install-ingress-nginx-controller/README.md)
* [Prometheus Stack](./doc/install-prometheus-stack/README.md)
* [Loki](./doc/install-loki/README.md)
* [Promtail](./doc/install-promtail/README.md)
* [Redis](./doc/install-redis/README.md)
* [PostgreSql](./doc/install-postgresql/README.md)
* [PgAdmin 4](./doc/install-pgAdmin4/README.md)
* [Minio](./doc/install-minio/README.md)
* [GitLab](./doc/install-gitlab/README.md)
* [GitLab Runner](./doc/install-gitlab-runner/README.md)
* [GitLab Shell (Настройка ssh)](./doc/install-gitlab-shell/README.md)

---

### Дополнительные ссылки
* [API Kubernetes 1.29](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.29/)
* [OpenLens](https://github.com/MuhammedKalkan/OpenLens/releases?ysclid=lrnfe9guk2158344056)
* [kubectl справочник](https://kubernetes.io/docs/reference/kubectl/generated/)
* [Шпаргалка по kubectl](https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/)
* [Решение проблем с Loki и Promtail (на китайском)](https://www.jianshu.com/p/6b24340c2cf1)
* [Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
* [Debug Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)
* [GitLab Создание секретов](https://docs.gitlab.com/charts/installation/secrets#initial-root-password)
* [Scale PHP-FPM](https://kamrul.dev/scale-php-fpm-on-kubernetes-with-keda/)