# docker build -t ansible-host .
# docker run -p 22:22 --name ansible-host-1 -it -d ansible-host
# docker exec -it ansible-host-1 bash
# https://hub.docker.com/r/rastasheep/ubuntu-sshd/dockerfile
FROM ubuntu:18.04

USER root
RUN apt-get update && \ 
    apt-get install -y openssh-server vim && \
    useradd -m -s /bin/bash ansible-user && \
    # echo 'ansible-user:yourpasswordgoeshere' | chpasswd && \ ### We technically should not need a password for this account
    mkdir /var/run/sshd && \
    sed -ri 's/^#?UsePAM\s+.*/UsePAM no/' /etc/ssh/sshd_config && \
    sed -ri 's/^#?ChallengeResponseAuthentication\s+.*/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config && \
    sed -ri 's/^#?PasswordAuthentication\s+.*/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin no/' /etc/ssh/sshd_config && \
    echo 'AllowUsers ansible-user' >> /etc/ssh/sshd_config

ADD authorized_keys /home/ansible-user/.ssh/authorized_keys

RUN chown ansible-user:ansible-user /home/ansible-user/.ssh && \ 
    chmod 700 /home/ansible-user/.ssh && \ 
    chown ansible-user:ansible-user /home/ansible-user/.ssh/authorized_keys && \ 
    chmod 600 /home/ansible-user/.ssh/authorized_keys && \
    usermod -aG sudo ansible-user

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]



