systemctl satatus bashible
/var/lib/bashible/bundle_steps
/var/lib/bashible/cleanup_static_node.sh - скрипт очищающий ноду от Deckhouse
### Можно создавать свои скрипты:
'''yaml
apiVersion: deckhouse.io/v1alpha1
kind: NodeGroupConfiguration
metadata:
  name: install-screenfetch.sh
spec:
  content: |
    bb-apt-install 'screenfetch'
    screenfetch
  nodeGroups:
  - "*"
  bundles:  
  - debian
  - ubuntu-lts
  weight: 90
'''
'''yaml
kubectl apply -f ngc.yaml
'''
'''yaml
journalctl -u bashible --since "1 hour ago"
'''