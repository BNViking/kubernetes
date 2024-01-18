[[Перейти в начало](../README.md)]

---

## Настройка haProxy в роле балансировщика

* [HAProxy Configuration Basics: Load Balance Your Servers](https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers)

Для того что бы иметь множество узлов Control Plane, необходимо создать балансировщика, который будет распределять трафик по узлам API Kubernetes

1. Установка haProxy (можно установить на 1 узел control-01.lan, так как у нас тестовый стенд)
   ```
   sudo apt install haproxy;
   ```
2. Редактируем файл с настройкой haProxy, в итоге получаем [/etc/haproxy/haproxy.cfg](./config/haproxy.cfg)

   *Все что добавляется*
   ```apacheconf
   frontend k8s-control-front
       bind 192.168.1.12:7443 #ClusterConfiguration.controlPlaneEndpoint: 192.168.1.12:7443
       default_backend k8s-control-backend
   ```
   ```apacheconf
   backend k8s-control-backend
       balance roundrobin
       server control-01 192.168.1.12:6443 check
       server control-02 192.168.1.13:6443 check
       server control-03 192.168.1.14:6443 check
   ```
   ```apacheconf
   listen stats
       bind *:9000
       mode http
       stats enable  #Включение статичтики
       stats hide-version  #Скрыть версию HAProxy
       stats realm Haproxy\ Statistics  #Заголовок окна
       stats uri /haproxy  #Адрес статистики
   ```
   
4. Запускаем haProxy
   ```bash
   sudo systemctl enable haproxy;
   sudo systemctl restart haproxy;
   sudo systemctl status haproxy;
   ```

Можно перейти по адресу http://192.168.1.12:9000/haproxy или http://control-01.lan:9000/haproxy и увидеть статистику
![haproxy_stats.png](img%2Fhaproxy_stats.png)

*Так как кластер не создан, control-01, control-02, control-03 будут отображаться как недоступные*
  
---

[[Перейти в начало](../README.md)]