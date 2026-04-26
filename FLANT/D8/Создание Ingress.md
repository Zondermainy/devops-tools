BareMetal:
### 1) Deckhouse - консоль -> Сеть -> Балансировка -> Ingress-контроллеры
### 2) HostPort:
Включить: валидация ingress-правил, валидация аннтоаций ingress-правил
Количество реплик для Baremetal неактуальный параметр
Указать лейблы, на группу каких узлов накинуть этот ингресс
```yaml
node.deckhouse.io/group: frontend
```
Список CIDR, которым разрешено подключаться к контроллеру
78.153.139.45/32 - можно только ему


### 1) Deckhouse - Глобальные настройки:
Ключи пользовательских toleration node-role
### 2) Cloud - консоль -> Сеть -> Балансировка -> Ingress-контроллеры
### 3) LoadBalancer, нужно в настройках ингресса:
Селектор узлов, на которых нужно разместить поды контроллера
node.deckhouse.io/group: app-frontend

Tolerations node-role Equal app-frontend NoExecute

### 4) В параметрах группы узлов:
Тейнты Лейблы и выражени node-role app-frontend NoExecute

