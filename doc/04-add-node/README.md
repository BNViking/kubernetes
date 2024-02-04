[[Перейти в начало](../../README.md)]

---

## Добавление узлов в Kubernetes (kubeadm)

Сначала надо убедиться, действует ли еще наш токен для добавления узлов

Вывод всех токенов

```bash
kubeadm token list
```

## Токен еще действителен

После того как будет развернут первый узел (*[Создание первого узла Control Plane (kubeadm)](../03-first-control-plane/README.md)*), унас будет 2 команды для добавления узлов:
 
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

Тут все просто - заходим на каждый узел и выполняем команду в зависимости от роли узла (Control/Worker)

## Время токена истекло

### Добавление узла Control Plane

Когда время токена истекает, соответственно команда для добавления узла перестает работать, но это не проблема, 
все что нам нужно - получить команду вида:
```bash
sudo kubeadm join PATH --token TOKEN \
  --discovery-token-ca-cert-hash sha256:CA_HASH \
  --certificate-key CA_KEY \
  --control-plane
```

Получаем PATH
```bash
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}' | cut -c9-
```
```
Результат: 192.168.1.12:7443
```

Получаем TOKEN
```bash
kubeadm token create
```
```
Результат: 755e5h.9mhg085l4gifma1p
```

Получаем CA_HASH
```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | \
openssl rsa -pubin -outform der 2>/dev/null | \
openssl dgst -sha256 -hex | sed 's/^.* //'
```
```
Результат: 980ea382fd4d42d926a4183b05fc1269e66472e9a63c9535c8c383dffce06f0f
```

Получаем CA_KEY
```bash
sudo kubeadm init phase upload-certs --upload-certs | tail -1
```
```
Результат: 758ad9ecdf96ea4df5b77975cd87a222a093d2d8f628bea72b22cffd873658c9
```

Подставляем полученные значения в команду, и добавляем узлы.
```bash
sudo kubeadm join 192.168.1.12:7443 --token 755e5h.9mhg085l4gifma1p \
  --discovery-token-ca-cert-hash sha256:980ea382fd4d42d926a4183b05fc1269e66472e9a63c9535c8c383dffce06f0f \
  --certificate-key 758ad9ecdf96ea4df5b77975cd87a222a093d2d8f628bea72b22cffd873658c9 \
  --control-plane
```

### Добавление узлов worker

Надо получить команду вида:
```bash
sudo kubeadm join PATH --token TOKEN \
  --discovery-token-ca-cert-hash sha256:CA_HASH 
```

Получаем PATH
```bash
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}' | cut -c9-
```
```
Результат: 192.168.1.12:7443
```

Получаем TOKEN
```bash
kubeadm token create
```
```
Результат: ta02xs.l69kmc5ypevxuiqo
```

Получаем CA_HASH
```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | \
openssl rsa -pubin -outform der 2>/dev/null | \
openssl dgst -sha256 -hex | sed 's/^.* //'
```
```
Результат: 6055f2a94b90184dd14f41cfe933b7379aea46b1de2c663564ecfb0ed80a86bc
```

В результате получаем команду
```bash
sudo kubeadm join 192.168.1.12:7443 --token ta02xs.l69kmc5ypevxuiqo \
  --discovery-token-ca-cert-hash sha256:6055f2a94b90184dd14f41cfe933b7379aea46b1de2c663564ecfb0ed80a86bc
```


---
*Если получили ошибку или добавили узел не с той ролью, необходимо запустить команду сброса, и снова добавить*
   ```bash
   sudo kubeadm reset
   ```
   ```bash
   sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X && sudo ipvsadm --clear;
   ```
---

### Смотрим какие узлы были добавлены

```bash
kubectl get nodes -o wide
```

```
   NAME             STATUS     ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
   control-01.lan   NotReady   control-plane   9m51s   v1.29.0   192.168.1.12   <none>        Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.2
   control-02.lan   NotReady   control-plane   2m6s    v1.29.0   192.168.1.13   <none>        Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.2
   control-03.lan   NotReady   control-plane   2m1s    v1.29.0   192.168.1.14   <none>        Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.2
   worker-01.lan    NotReady   <none>          2m11s   v1.29.0   192.168.1.15   <none>        Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.2
   worker-02.lan    NotReady   <none>          66s     v1.29.0   192.168.1.16   <none>        Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.2
   worker-03.lan    NotReady   <none>          60s     v1.29.0   192.168.1.17   <none>        Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.2
```

*На данный момент статус NotReady, но после установки CNI все будет ОК =)*

---

[[Перейти в начало](../../README.md)]