# 100 - Installing k3s on CentOS 7

Based on "k3s.io/k3s" at https://github.com/k3s-io/k3s/issues/1019#issuecomment-593043089

I ([Lohann Paterno Coutinho Ferreira](https://github.com/Lohann)) finally got K3S working on Centos 7 and docker, it wasn't working even after replace firewalld by iptables.

Facts:
•	K3S doesn't support firewalld (see #1371)
•	Docker containers can't connect to localhost on ***Docker-for-Linux***, after a lot debugging I realized that my iptables rules was dropping the docker container packages, it trigger the Crash Loop because local-path-provisioner and metrics-server needs to connect to k3s on internal host.

So I just added following rule to iptables in order to fix it:

```
$ iptables -A INPUT -s 10.42.0.0/16 -d <host_internal_ip>/32 -j ACCEPT
```

Where 10.42.0.0/16 is the default k3s network CIDR, change it if you defined a different value on --cluster-cidr flag.

## The Step-by-Step Solution:

### Stop and remove k3s (if k3s was previously installed, otherwise skip this step)
```
$ k3s-killall.sh
$ k3s-uninstall.sh
```

### Stop and remove all k3s containers (if k3s was previously installed, otherwise skip this step)
```
docker stop $(docker ps -a -q --filter "name=k8s_") | xargs docker rm
```

### Replace firewalld by iptables
```
$ systemctl stop firewalld
$ systemctl disable firewalld
$ yum install iptables-services
$ systemctl start iptables
$ systemctl enable iptables
```

Configure iptables rules, my setup is based on this article: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-basic-iptables-firewall-on-centos-6

### Erase iptables rules
```
$ iptables -F
```

### Block the most common attacks
```
$ iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
$ iptables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
$ iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
```

### Enable outgoing connections
```
$ iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
$ iptables -A INPUT -i lo -j ACCEPT
$ iptables -P OUTPUT ACCEPT
```

### Open Traefik http and https ports
```
$ iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
$ iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
```

### Open SSH port
```
$ iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
```

### Block everything else
```
$ iptables -P INPUT DROP
```

### Here is where the magic happens, enable connections from k3s pods to your host internal ip:
```
$ iptables -A INPUT -s 10.42.0.0/16 -d <host_internal_ip>/32 -j ACCEPT
```

### Save changes 
```
$ iptables-save | sudo tee /etc/sysconfig/iptables
```

### Restart iptables
```
service iptables restart
```

### Install and run k3s (will auto-start after installation)
```
curl -sfL https://get.k3s.io | sh -s 
```
