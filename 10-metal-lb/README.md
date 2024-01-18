[[Перейти в начало](../README.md)]

---

## Настройка MetalLB

Настроим, что бы сервис типа LoadBalancer получал внешний IP из диапазона 192.168.1.180-192.168.1.185

Далее уже в нашем балансировщике ([haProxy](../02-haProxy/README.md)) можно указать, 
что все запросы на `*.bnvkube.lan` можно отправлять на этот список IP.

1. [Устанавливаем MetalLB](../install-metal-lb/README.md)
2. Редактируем файл [./00-metal-lb-ip-address-pool.yaml](./00-metal-lb-ip-address-pool.yaml)
    ```yaml
    ---
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
      name: pool-ip
      namespace: metallb
    spec:
      # Список IP адресов которые будут назначаться
      # Можно использовать маску, например 192.168.1.0/24
      addresses:
        - 192.168.1.180-192.168.1.185  
    ---
    apiVersion: metallb.io/v1beta1
    kind: L2Advertisement
    metadata:
      name: base
      namespace: metallb
    spec:
      ipAddressPools:
        - pool-ip
    ```

3. Применяем манифест
    ```bash
    kubectl apply -f ./00-metal-lb-ip-address-pool.yaml
    ```

---

[[Перейти в начало](../README.md)]