
FROM ubuntu:latest

# Install core components
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
    apt-get dist-upgrade -y && apt-get autoremove -y && apt-get clean && \
    apt-get install -y software-properties-common && \
    apt-get install -y postfix && \
    apt-get install -y mysql-client curl gcc git gnupg-agent \
        make python openssl redis-server sudo vim zip locales wget iproute2 supervisor

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8

RUN useradd misp && usermod -aG sudo misp
	
# Install script
COPY INSTALL_NODB.sh .
RUN chmod +x INSTALL_NODB.sh
RUN echo "ALL ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers


USER misp

RUN bash INSTALL_NODB.sh -A -u

USER root

# Supervisord Setup
RUN ( \
    echo '[supervisord]'; \
    echo 'nodaemon = true'; \
    echo ''; \
    echo '[program:postfix]'; \
    echo 'process_name = master'; \
    echo 'directory = /etc/postfix'; \
    echo 'command = /usr/sbin/postfix -c /etc/postfix start'; \
    echo 'startsecs = 0'; \
    echo 'autorestart = false'; \
    echo ''; \
    echo '[program:redis-server]'; \
    echo 'command=redis-server /etc/redis/redis.conf'; \
    echo ''; \
    echo '[program:apache2]'; \
    echo 'command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -D FOREGROUND"'; \
    echo ''; \
    echo '[program:resque]'; \
    echo 'command=/bin/bash /var/www/MISP/app/Console/worker/start.sh'; \
    echo 'user = www-data'; \
    echo 'startsecs = 0'; \
    echo 'autorestart = false'; \
    echo ''; \
    echo '[program:misp-modules]'; \
    echo 'command=/bin/bash -c "/var/www/MISP/venv/bin/misp-modules -l 127.0.0.1 -s"'; \
    echo 'user = www-data'; \
    echo 'startsecs = 0'; \
    echo 'autorestart = false'; \
) >> /etc/supervisor/conf.d/supervisord.conf

# Add run script
# Trigger to perform first boot operations
ADD run.sh /run.sh
RUN chmod 0755 /run.sh && touch /.firstboot.tmp
# Make a backup of /var/www/MISP to restore it to the local moint point at first boot
WORKDIR /var/www/MISP
RUN tar czpf /root/MISP.tgz .

VOLUME /var/www/MISP
EXPOSE 80
ENTRYPOINT ["/run.sh"]
