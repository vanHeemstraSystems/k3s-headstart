FROM centos:7

# The following instructions are required for k3s to install successfully on CentOS 7

# Copy Docker systemctl replacement 
# See also: github.com/gdraheim/docker-systemctl-replacement/files/docker/systemctl.py
COPY systemctl.py /usr/bin/systemctl 
RUN chmod a+x /usr/bin/systemctl

# Replace firewalld by iptables
# RUN systemctl stop firewalld # SKIP: Not required and throws an error
# RUN systemctl disable firewalld # SKIP: Not required and throws an error
RUN yum -y install iptables-services
RUN systemctl start iptables
RUN systemctl enable iptables

# Erase iptables rules
RUN iptables -F

# Block the most common attacks
RUN iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
RUN iptables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
RUN iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# Enable outgoing connections
RUN iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
RUN iptables -A INPUT -i lo -j ACCEPT
RUN iptables -P OUTPUT ACCEPT

# Open Traefik http and https ports
RUN iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
RUN iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT

# Open SSH port
RUN iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT

# Block everything else
RUN iptables -P INPUT DROP

# Here is where the magic happens, enable connections from k3s pods to your host internal ip
# Get host internal ip without trailing spaces
RUN HOST_INTERNAL_IP=`echo $(hostname -I) | sed 's/ *$//g'`
# Run iptables -A INPUT -s 10.42.0.0/16 -d <host_internal_ip>/32 -j ACCEPT
RUN iptables -A INPUT -s 10.42.0.0/16 -d $HOST_INTERNAL_IP/32 -j ACCEPT

# Save changes
RUN iptables-save | tee /etc/sysconfig/iptables

# Restart iptables
RUN service iptables restart

# Install and run k3s (will auto-start after installation)
RUN curl -sfL https://get.k3s.io | sh -s 

# Verify k3s installation
RUN systemctl status k3s

# Change file permissions of k3s.yaml
RUN chmod 644 /etc/ranger/k3s/k3s.yaml

# Verify installation of kubectl (comes automatically with k3s installation)
RUN kubectl --help > /dev/null

# Force the container to keep running, for testing only!
ENTRYPOINT ["tail", "-f", "/dev/null"]
