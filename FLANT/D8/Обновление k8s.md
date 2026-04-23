### 1) Определяем, какой под d8 является лидером:
'''yaml
kubectl -n d8-system get po -l leader=true
'''

### 2) Проваливаемся в под в режим редактирования:
'''yaml
kubectl -n d8-system exec -ti deckhouse-b9b86d59c-46ssb -c deckhouse -- deckhouse-controller edit cluster-configuration
'''

### 3) Меняем строчку kubernetesVersion на какую-либо определённую
'''yaml
kubernetesVersion: Automatic
'''