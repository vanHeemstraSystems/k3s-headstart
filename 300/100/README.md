# 100 - Installing k3s on CentOS 7

Based on "k3s.io/k3s" at https://github.com/k3s-io/k3s/issues/1019#issuecomment-593043089

I ([Lohann Paterno Coutinho Ferreira](https://github.com/Lohann)) finally got K3S working on Centos 7 and docker, it wasn't working even after replace firewalld by iptables.

Facts:

-	K3S doesn't support firewalld (see #1371)
-	Docker containers can't connect to localhost on ***Docker-for-Linux***, after a lot debugging I realized that my iptables rules was dropping the docker container packages, it trigger the Crash Loop because local-path-provisioner and metrics-server needs to connect to k3s on internal host.

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

### Verify k3s installation

After the installation is completed, you can check if the k3s service is running by executing the below command.
```
$ sudo systemctl status k3s
```

It will show you something like this:
```
* [GREEN DOT] k3s.service - Lightweight Kubernetes
   Loaded: loaded (/etc/systemd/system/k3s.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-06-10 11:51:19 UTC; 27min ago
     Docs: https://k3s.io
  Process: 4768 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
  Process: 4763 ExecStartPre=/sbin/modprobe br_netfilter (code=exited, status=0/SUCCESS)
 Main PID: 4771 (k3s-server)
    Tasks: 9
   Memory: 668.9M
   CGroup: /system.slice/k3s.service
           └─4771 /usr/local/bin/k3s server

Jun 10 12:18:48 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:48.832003377Z" level=info msg="...nd"
Jun 10 12:18:49 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:49.837054124Z" level=info msg="...nd"
Jun 10 12:18:50 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:50.841306661Z" level=info msg="...nd"
Jun 10 12:18:51 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:51.695300380Z" level=info msg="...te"
Jun 10 12:18:51 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:51.845131095Z" level=info msg="...nd"
Jun 10 12:18:51 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:51.877252567Z" level=info msg="...TC"
Jun 10 12:18:52 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:52.038675857Z" level=info msg="...TC"
Jun 10 12:18:52 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:52.045865380Z" level=error msg="Fa...
Jun 10 12:18:52 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:52.929377690Z" level=info msg="...nd"
Jun 10 12:18:53 555642125a1c.mylabserver.com k3s[4771]: time="2021-06-10T12:18:53.933513681Z" level=info msg="...nd"
Hint: Some lines were ellipsized, use -l to show in full.
```

## Change file permissions of k3s.yaml
```
$ cd /etc/ranger/k3s
$ chmod 644 k3s.yaml
```

## Verify installation of kubectl (comes automatically with k3s installation)
```
$ kubectl --help
```
