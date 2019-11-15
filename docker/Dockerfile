############################################################
## Manual commands:
## docker build -t ansible-host .
## docker run -p 22:22 --name ansible-host-1 -it -d ansible-host
## docker exec -it ansible-host-1 bash
############################################################
## Originally from: 
## https://hub.docker.com/r/rastasheep/ubuntu-sshd/dockerfile
############################################################

FROM ubuntu:18.04
USER root

RUN apt-get update && \ 
    apt-get install -y openssh-server vim

RUN useradd -m -s /bin/bash ansible-user && \
    # echo 'ansible-user:yourpasswordgoeshere' | chpasswd && \ ### We technically should not need a password for this account
    mkdir /var/run/sshd && \
    echo 'AllowUsers ansible-user' >> /etc/ssh/sshd_config

ADD authorized_keys /home/ansible-user/.ssh/authorized_keys

RUN chown ansible-user:ansible-user /home/ansible-user/.ssh && \ 
    chmod 700 /home/ansible-user/.ssh && \ 
    chown ansible-user:ansible-user /home/ansible-user/.ssh/authorized_keys && \ 
    chmod 600 /home/ansible-user/.ssh/authorized_keys && \
    usermod -aG sudo ansible-user

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]



