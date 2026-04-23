## Бекап etcd и восстановление

https://deckhouse.ru/modules/control-plane-manager/faq.html#%D0%BA%D0%B0%D0%BA-%D1%81%D0%B4%D0%B5%D0%BB%D0%B0%D1%82%D1%8C-%D1%80%D0%B5%D0%B7%D0%B5%D1%80%D0%B2%D0%BD%D1%83%D1%8E-%D0%BA%D0%BE%D0%BF%D0%B8%D1%8E-etcd-%D0%B2%D1%80%D1%83%D1%87%D0%BD%D1%83%D1%8E
Ссылка в документации на бекап etcd

### 1) Скрипт создания бекапа etcd

'''yaml
#!/usr/bin/env bash
set -e

pod=etcd-`hostname`
d8 k -n kube-system exec "$pod" -- /usr/bin/etcdctl --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key --endpoints https://127.0.0.1:2379/ snapshot save /var/lib/etcd/${pod##*/}.snapshot && \
mv /var/lib/etcd/"${pod##*/}.snapshot" etcd-backup.snapshot && \
cp -r /etc/kubernetes/ ./ && \
tar -cvzf kube-backup.tar.gz ./etcd-backup.snapshot ./kubernetes/
rm -r ./kubernetes ./etcd-backup.snapshot
'''

### 2) Скрипт проверки членов etcd
'''yaml
#!/usr/bin/env bash
set -e

pod=etcd-`hostname`
d8 k -n kube-system exec "$pod" -- /usr/bin/etcdctl --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key --endpoints https://127.0.0.1:2379/ member list
'''

### 3) Перед восстановлением etcd из бекапа необходимо сделать кластер с одним master

'''yaml
kubectl delete node zharnikov-intern-master2
'''
### 4)Заходим на ноду zharnikov-intern-master2 и зачищаем её

'''yaml
cd /var/lib/bashible
bash cleanup_static_node.sh 
#попросят выполнить команду с флагом
'''

### 5) Начинаем процесс восстановления:
Качаем etcd с утилитой etcdutl: https://github.com/etcd-io/etcd/
'''yaml
 tar -xzvf etcd-v3.6.10-linux-amd64.tar.gz && mv etcd-v3.6.10-linux-amd64/etcdutl /usr/local/bin/etcdutl
'''

### 6) Отключаем etcd (просто переносим staticpod куда-нибудь)
'''yaml
mv /etc/kubernetes/manifests/etcd.yaml ~/etcd.yaml
'''
kubeapi отвалится, но проверить работоспособность подов можно через crictl ps

### 7) Дополнительно бекапим перед бекапом, чтобы не сделать ещё хуже:
'''yaml
cp -r /var/lib/etcd/member/ /var/lib/deckhouse-etcd-backup
'''

### 8) Зачищаем /var/lib/etcd/member/
'''yaml
rm -rf /var/lib/etcd/member/
'''

### 9) Расспаковываем бекап:
'''yaml
tar -xvf kube-backup.tar.gz
'''

### 10) Восстанавливаем снапшот etcd
'''yaml
ETCDCTL_API=3 etcdutl snapshot restore ~/etcd-backup.snapshot --data-dir=/var/lib/etcd
'''

### 11) Перемещаем манифест etcd обратно
'''yaml
mv ~/etcd.yaml /etc/kubernetes/manifests/etcd.yaml
'''

### 12) Рекомендация: ребутнуть все узлы кластера

### 13) Нужно реджойнуть мастера кластера 
'''yaml
kubectl delete no zharnikov-intern-master1
kubectl delete no zharnikov-intern-master2
kubectl -n d8-cloud-instance-manager get po
kubectl -n d8-cloud-instance-manager delete po bashible-apiserver-c895f66d7-44srb
kubectl -n d8-cloud-instance-manager delete po bashible-apiserver-c895f66d7-v9jq4
(удаляем поды в статусе pending)
''' 

### 14) Запускаем bootstrap.sh снова на узлах
(нужно дождать пока control plane manager полностью будет готов, чтобы скрипты успели обновиться)

'''yaml
kubectl -n d8-cloud-instance-manager get secret manual-bootstrap-for-master -ojsonpath="{.data.bootstrap\.sh}"; echo

echo #СЮДА СКРИПТ# | base64 -d | sudo bash
'''
