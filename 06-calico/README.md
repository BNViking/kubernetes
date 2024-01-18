[[Перейти в начало](../README.md)]

---

## Установка CNI 

[Устанавливаем Calico](../install-calico/README.md)

### Исправляем coredns

*Перезапускаем coredns, что бы они разъехались по разным узлам*
```bash
kubectl -n kube-system rollout restart deployment coredns
```

Проверяем
```bash
kubectl -n kube-system get pods -o wide | grep coredns
```

```
NAME                                     READY   STATUS    RESTARTS      AGE    IP             NODE             NOMINATED NODE   READINESS GATES
coredns-85469dc56b-b5b7g                 1/1     Running   0             78s   10.20.98.195   worker-01.lan    <none>           <none>
coredns-85469dc56b-p8vnd                 1/1     Running   0             78s   10.20.37.194   worker-02.lan    <none>           <none>
```

---

[[Перейти в начало](../README.md)]