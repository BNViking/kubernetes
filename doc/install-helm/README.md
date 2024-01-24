[[Перейти в начало](../../README.md)]

---

## Установка Helm

* [Инструкция на официальном сайте](https://helm.sh/docs/intro/install/)

---

### Debian/Ubuntu

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
```

```bash
sudo apt-get install apt-transport-https --yes
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install helm
```

---

### Windows

Скачиваем [архив](https://get.helm.sh/helm-v3.13.3-windows-amd64.zip) (версия может измениться, ссылку на архив можно взять из https://github.com/ScoopInstaller/Main/blob/master/bucket/helm.json)

Распаковываем, закидываем в любую папку, добавляем в переменные среды PATH

---
Проверка версии
```bash
helm version
```
```
version.BuildInfo{Version:"v3.13.3", GitCommit:"c8b948945e52abba22ff885446a1486cb5fd3474", GitTreeState:"clean", GoVersion:"go1.20.11"}
```
---

[[Перейти в начало](../../README.md)]