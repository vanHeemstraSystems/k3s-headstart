FROM centos:7

# The following instructions are required for k3s to install successfully on CentOS 7

# Copy Docker systemctl replacement 
# See also: github.com/gdraheim/docker-systemctl-replacement/files/docker/systemctl.py
COPY systemctl.py /usr/bin/systemctl 
RUN chmod a+x /usr/bin/systemctl

# Replace firewalld by iptables
# RUN systemctl stop firewalld
# RUN systemctl disable firewalld
# RUN yum -y install iptables-services
# RUN systemctl start iptables
# RUN systemctl enable iptables




# More code goes here ...

# Force the container to keep running, for testing only!
ENTRYPOINT ["tail", "-f", "/dev/null"]
