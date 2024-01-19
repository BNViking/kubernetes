[[Перейти в начало](../README.md)]

---

## Логирование

Для логирования будем использовать связку Loki + Promtail

1. Устанавливаем [Loki](../install-loki/README.md)
2. Устанавливаем [Promtail](../install-promtail/README.md)

После установки переходим в [grafana.bnvkube.lan](https://grafana.bnvkube.lan)\
В настройках добавляем Новое соединение, выбираем Loki\
В URL вписываем `http://loki-gateway.logging`, сохраняем.


---

[[Перейти в начало](../README.md)]