[[Перейти в начало](../README.md)]

---

## Подготовка узла 

* [Рекомендации от kubernetes.io](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

*Все действия необходимо провести на каждый машине, которая будет в узле kubernetes.
Если не созданы все узлы, целесообразно сделать 1 машину и потом сделать ее клон для остальных узлов.*

1. Делаем обновление, устанавливаем openssh-server
   ```bash
   sudo apt update;
   sudo apt full-upgrade -y;
   sudo apt install openssh-server -y;
   ```
   
2. Отключаем swap
   ```bash
   sudo swapoff -a;
   sudo rm -rf /swap.img;
   ```
   
3. Отключаем монтирование раздела для swap
   ```bash
   sudo sed -i '/swap.img/ s/^/#/' /etc/fstab;
   ```
   
4. Отключаем пользователю ввод пароля для sudo (необязательно)
   ```bash
   echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER;
   ```
   
5. Добавляем репозиторий kubernetes
   ```bash
      curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg;
      echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list;
   ```
   
6. Устанавливаем необходимые пакеты
   ```bash
   sudo apt-get update;
   sudo apt install -y kubeadm kubelet kubectl kubernetes-cni containerd apt-transport-https ca-certificates curl gnupg nfs-common ipvsadm;
   ```
   
7. Отключаем firewall
   ```bash
   sudo ufw disable;
   ```
   
8. Настройка сети. Создаем/редактируем файл /etc/sysctl.d/99-kubernetes-cri.conf и добавляем в него
   ```bash
   net.ipv4.ip_forward=1
   net.ipv4.ip_nonlocal_bind=1
   net.bridge.bridge-nf-call-iptables=1
   #net.bridge.bridge-nf-call-ip6tables=1 #Использование сети ipv6
   ```
   
9. Включение модулей
   ```bash
   sudo modprobe overlay;
   sudo modprobe br_netfilter;
   ```
   
10. Добавление модулей в загрузку. Создаем/редактируем файл **/etc/modules-load.d/containerd.conf** и добавляем в него строки
   ```
   overlay
   br_netfilter
   ```
11. Применяем настройки systemd
   ```bash
   sudo sysctl --system;
   ```

12. Включаем в containerd работу с cgroup
   ```bash
   sudo mkdir /etc/containerd;
   containerd config default | sudo tee /etc/containerd/config.toml;
   sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml;
   ```

13. Добавляем настройки для crictl. Создаем/редактируем файл /etc/crictl.yaml, добавляем в него строки
   ```yaml
   runtime-endpoint: "unix:///run/containerd/containerd.sock"
   image-endpoint: "unix:///run/containerd/containerd.sock"
   timeout: 0
   debug: false
   pull-image-on-create: false
   disable-pull-on-run: false
   ```

14. Перезапускаем службы
   ```bash
   sudo service containerd restart;
   sudo service kubelet restart;
   ```

[[Перейти в начало](../README.md)]