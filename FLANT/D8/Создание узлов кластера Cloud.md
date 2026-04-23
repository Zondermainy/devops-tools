## Deckhouse умеет создавать виртуалки в cloud сам, bare metal - нет.

### CloudEphemeral:

## 1) Посмотреть текущее состояние узлов:
# - Консоль Deckhouse -> Узлы
# - Группы узлов (тут отображаются все типы узлов)

## 2) Создать класс машин:
# - Консоль Deckhouse -> Узлы
# - Классы машин -> Создать
# - Указать шаблон машины под нужды (large, small)

## 3) Создать Cloud Ephemeral тип узлов:
# - Консоль Deckhouse -> Узлы
# - Группы узлов
# - Создать Cloud Ephemeral
# - Сопоставить тип узла классу машин
# - Имя (например, monitoring)

deckhouse контролер пошёл в api клауда заказывать железо ->
# DECKHOUSE НАЧНЁТ САМ СОЗДАВАТЬ УЗЕЛ


## Cloud Permanent:
1. Подготовка окружения
Запуск контейнера с утилитой dhctl нужной версии. Монтирование SSH-ключей для доступа к мастер-ноде.
```yaml
docker run --pull=always -it \
  -v "$HOME/.ssh/:/tmp/.ssh/" \
  registry.deckhouse.ru/deckhouse/ee/install:v1.74.15 \
  bash
```
2. Проверка подключения (Connectivity Check)
Проверка доступности мастера кластера и корректности SSH-ключей перед внесением изменений.
```yaml
dhctl terraform check \
  --ssh-host 111.88.253.61 \
  --ssh-agent-private-keys /tmp/.ssh/education_id_rsa \
  --ssh-user ubuntu
```
3. Редактирование конфигурации провайдера
Открытие редактора для изменения конфигурации кластера (provider-cluster-configuration). Здесь описываются параметры инфраструктуры (ноды, сети, диски).
```yaml
dhctl config edit provider-cluster-configuration \
  --ssh-host 111.88.253.61 \
  --ssh-agent-private-keys /tmp/.ssh/education_id_rsa \
  --ssh-user ubuntu
```
Ключевой блок конфигурации (YAML):
Добавление или изменение группы узлов frontend.
```yaml
nodeGroups:
  - name: frontend           # Имя группы узлов
    replicas: 2              # Количество нод
    zones:                   # Зоны доступности Yandex Cloud
      - ru-central1-b
      - ru-central1-a
    instanceClass:           # Параметры виртуальных машин (Yandex Compute)
      cores: 2               # vCPU
      memory: 4096           # RAM в МБ (4 ГБ)
      imageID: fd83ica41cade1mj35sr # ID образа ОС (например, Ubuntu/Container Optimized)
      coreFraction: 20       # Гарантия производительности vCPU (20%)
      externalIPAddresses:   # Настройка внешних IP
        - Auto               # Автоматическое назначение публичного IP
        - "103.76.53.124"    # Или статический IP (если требуется)
```
4. Применение изменений (Converge)
Запуск процесса применения конфигурации. Deckhouse сравнит желаемое состояние (из YAML) с текущим и создаст/обновит ресурсы в облаке.

```yaml
dhctl converge \
  --ssh-host 111.88.253.61 \
  --ssh-agent-private-keys /tmp/.ssh/education_id_rsa \
  --ssh-user ubuntu
  ```