### Настройка taints & tolerations для распледения прометеуса только в мониторинг узлы:
```yaml
Группы узлов:
Лейблы группы узлов monitoring key node-role value monitoring
Тейнты node-role monitoring NoExecute

Prometheus:
Имя StorageClass - localpath-monitoring

Селектор узлов, на которых нужно размеcтить поды контроллера
key node.deckhouse.io/group
value monitoring

Tolerations node-role Equal monitoring NoExecute
```