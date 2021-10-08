# 300 - Install K3S Worker

Grab token from the master node to be able to add worked nodes to it:

```
cat /var/lib/rancher/k3s/server/node-token
```

Install k3s on the worker node and add it to our cluster:

```
curl -sfL https://get.k3s.io | K3S_NODE_NAME=k3s-worker-01 K3S_URL=https://<IP>:6443 K3S_TOKEN=<TOKEN> sh - 
```
