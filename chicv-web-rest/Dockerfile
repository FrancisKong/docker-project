FROM ubuntu:16.04

RUN useradd chicv -p chicv -s /bin/bash
MAINTAINER chicv-web-rest

ENV LANG=C.UTF-8
ARG DEBIAN_FRONTED=noninteractive
RUN   apt-get clean \
   && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 40976EAF437D05B5  \
   && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3B4FE6ACC0B21F32 \
   && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys  4F4EA0AAE5267A6C \
   && apt-get update \
   && apt-get upgrade -y \
   && apt-get install aptitude bash-completion -y \
   && aptitude install sudo \
   && echo 'chicv   ALL=(ALL)  NOPASSWD:ALL' >> /etc/sudoers \
   && apt-get install apt-utils -y \
   && apt-get install software-properties-common python-software-properties -y \
   && apt-get install apt-transport-https -y \
   && add-apt-repository ppa:ondrej/php \
   && apt-get update


RUN apt-get install php7.1 php7.1-common  -y --allow-unauthenticated \
 && apt-get install php7.1-curl php-redis php7.1-xml php7.1-zip php7.1-gd php7.1-mysql php7.1-mbstring -y --allow-unauthenticated \
 && apt-get install php7.1-fpm php7.1-cli php7.1-interbase php7.1-json php7.1-intl  -y --allow-unauthenticated  \
 && apt-get install php7.1-opcache php7.1-mcrypt php7.1-readline  php-msgpack -y  --allow-unauthenticated \
 && apt-get install nginx composer dnsmasq -y \
 && apt-get install vim -y \
 && apt-get install -y net-tools

ENV ELASTICSEARCH_VERSION 2.4.4
ENV ELASTICSEARCH_DEB_VERSION 2.4.4

RUN  apt-get install wget git -y \
  && wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | apt-key add - \
  && echo 'deb http://packages.elasticsearch.org/elasticsearch/2.x/debian stable main' > /etc/apt/sources.list.d/elasticsearch.list \
  && apt-get update \
  && apt-get install elasticsearch openjdk-8-jre -y \
  && mkdir /home/chicv/logs/www-log -p \
  && chown chicv:chicv /home/chicv/ -R \
  && chmod 755 /home/chicv \
  && chmod 777 /home/chicv/logs -R

RUN echo "mysql-server mysql-server/root_password password chicv" | debconf-set-selections \
   && echo "mysql-server mysql-server/root_password_again password chicv" | debconf-set-selections \
   && apt-get -y install mysql-server \
   &&  apt-get install redis-server influxdb influxdb-client -y

COPY  my.cnf  /etc/mysql/my.cnf
COPY  nginx.conf  /etc/nginx/nginx.conf
COPY  elasticsearch.yml  /etc/elasticsearch/elasticsearch.yml
# COPY  jfn.tar.gz  /home/chicv/jfn.tar.gz
COPY  bashrc  /home/chicv/.bashrc
RUN sed -i "s/error_log = \/var\/log\/php7.1-fpm.log/error_log = \/home\/chicv\/logs\/php7-error.log/" /etc/php/7.1/fpm/php-fpm.conf \
    && sed -i "s/^error_reporting\s=\sE_ALL\s&\s~E_DEPRECATED\s&\s~E_STRICT$/error_reporting = E_ALL/g" /etc/php/7.1/fpm/php.ini \
    && sed -i "s/^display_errors = Off$/display_errors = On/g" /etc/php/7.1/fpm/php.ini \
    && sed -i "s/user = www-data/user = chicv/" /etc/php/7.1/fpm/pool.d/www.conf \
    && sed -i "s/group = www-data/group = chicv/" /etc/php/7.1/fpm/pool.d/www.conf \
    && sed -i "s/listen.owner = www-data/listen.owner = chicv/" /etc/php/7.1/fpm/pool.d/www.conf \
    && sed -i "s/listen.group = www-data/listen.group = chicv/" /etc/php/7.1/fpm/pool.d/www.conf \
      && usermod -d /var/lib/mysql/ mysql \
    && sed -i /\ SCRIPT_FILENAME\ /d /etc/nginx/fastcgi_params \
    && echo "nameserver 8.8.8.8" > /etc/resolv.dnsmasq \
    && service influxdb start \
    && sleep 3 \
    && influx --execute "CREATE DATABASE stylewe" \
    && influx --execute "CREATE RETENTION POLICY autogen ON stylewe DURATION 1d REPLICATION 1" \
    && service influxdb stop \
    && sed -i "s/^resolv-file.*$/resolv-file=\/etc\/resolv.dnsmasq/" /etc/dnsmasq.conf  \
      && echo "conf-dir=/home/chicv/Cmd/dnsmasq.d/,*.conf" >>  /etc/dnsmasq.conf \
    && chown chicv:chicv /home/chicv -R
 #   && mkdir  /var/run/mysqld && chown -R mysql:mysql /var/lib/mysql /var/run/mysqld
#RUN service mysql start \
    # &&  mysql -uroot -pchicv -e 'CREATE USER stylewe IDENTIFIED BY "stylewe";GRANT ALL PRIVILEGES ON *.* TO stylewe@"%";create database stylewe DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;flush privileges;'\
    # &&  echo "解压数据" \
    # &&  tar -zxf /home/chicv/jfn.tar.gz -C /home/chicv/ \
    # &&  echo "start import data into database" \
    # &&  mysql -ustylewe -pstylewe stylewe < /home/chicv/jfn.sql \
    # && rm -rf /home/chicv/jfn* \
    # && service mysql stop


#RUN echo "03 */4 * * * /home/chicv/Cmd/auto-pull.sh" > /home/chicv/tmpcron \
    #   && crontab /home/chicv/tmpcron \
    #&& rm -f /home/chicv/tmpcron
CMD /home/chicv/Cmd/startUp.sh  && /bin/bash
USER chicv
WORKDIR /home/chicv
RUN echo 'thanks for building'
