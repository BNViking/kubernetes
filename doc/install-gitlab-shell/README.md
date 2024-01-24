[[Перейти в начало](../../README.md)]

---

## Установка GitLab Shell (Настройка ssh)

---

### Полезные ссылки

* [GitLab Chart](https://gitlab.com/gitlab-org/charts/gitlab)
* [GitLab документация](https://docs.gitlab.com/charts/)
* [Конфигурация GitLab Shell](https://docs.gitlab.com/charts/charts/gitlab/gitlab-shell/)

---

### Установка зависимостей

* [GitLab](../install-gitlab/README.md)

---

После установки **GitLab** будет добавлен сервис **gitlab-gitlab-shell**, но так как нет проброса портов, 
то он работать не будет, поэтому надо настроить **ingress nginx controller**

_Перед настройкой добавьте **ssh** ключ пользователю в **GitLab**_

### Пошаговая инструкция

1. Изменяем конфигурацию **ConfigMap:ingress-nginx-controller**
   ```yaml
   data:
     annotation-value-word-blocklist: "load_module,lua_package,_by_lua,location,root,proxy_pass,serviceaccount,{,},',\""
     hsts: "true"
     hsts-include-subdomains: "false"
     hsts-max-age: "63072000"
     server-name-hash-bucket-size: "256"
     server-tokens: "false"
     ssl-ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4"
     ssl-protocols: "TLSv1.3 TLSv1.2"
     upstream-keepalive-connections: "100"
     upstream-keepalive-requests: "1000"
     upstream-keepalive-time: "30s"
     upstream-keepalive-timeout: "5"
     use-http2: "true"
   ```
2. Изменяем конфигурацию **Deployment:ingress-nginx-controller**
   ```yaml
   spec:
     containers:
       args:
          ...
          - '--tcp-services-configmap=gitlab/gitlab-nginx-ingress-tcp'
       ports:
         ...
         # Добавляем 22 порт
         - name: gitlab-shell
           containerPort: 22
           protocol: TCP
   ```

3. Изменяем конфигурацию **Service:ingress-nginx-controller**
   ```yaml
   spec:
     ports:
       ...
       #Добавляем 22 порт
       - name: gitlab-shell
         port: 22
         protocol: TCP
         targetPort: gitlab-shell
   ```
---

Проверка:
```bash
ssh git@gitlab.bnvkube.lan
```
```
Welcome to GitLab, @bnviking!
Connection to gitlab.bnvkube.lan closed.
```

---

[[Перейти в начало](../../README.md)]