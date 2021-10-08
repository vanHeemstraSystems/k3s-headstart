# 200 - Install K3S Master

```
curl -sfL https://get.k3s.io | sh -s - --no-deploy traefik --write-kubeconfig-mode 644 --node-name k3s-master-01
```
