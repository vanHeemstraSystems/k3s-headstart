# 400 - Minimal Setup (Optional)

Based on "Rancher's k3s - First steps" at https://gitlab.com/cloud-versity/rancher-k3s-first-steps

Video on "K3S (by Rancher) | Setup a lightweight Kubernetes Cluster in Minutes | Hands-on Tutorial" at https://www.youtube.com/watch?v=1hwGdey7iUU

***Additional information***

You can change the settings of k3s by changing the service settings e.g. with nano /etc/systemd/system/k3s.service.

Make sure to restart the service afterwards: 

```
systemctl restart k3s
```

In many cases you can just run the installer with different variables again and it will configure your cluster accordingly without deleting it in the first place.

## 100 - Sign up and Setup Environment

See [README.md](./100/README.md)

## 200 - Install K3S Master

See [README.md](./200/README.md)

## 300 - Install K3S Worker

See [README.md](./300/README.md)

## 400 - Install NGinX Ingress Controller

See [README.md](./400/README.md)

## 500 - Deploy Demo Application

See [README.md](./500/README.md)
