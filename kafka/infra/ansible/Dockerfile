# Shortcut for pre-installing packages to save time.
FROM geerlingguy/docker-centos7-ansible
ENV container docker

RUN yum -y install java-11-openjdk \
      openssl \
      curl \
      which \
      rsyslog \
      libselinux-python \
      unzip

#https://docs.confluent.io/platform/current/installation/installing_cp/rhel-centos.html#get-the-software
RUN echo $'[Confluent.dist]\n\
name=Confluent repository (dist)\n\
baseurl=https://packages.confluent.io/rpm/7.1/$releasever\n\
gpgcheck=1\n\
gpgkey=https://packages.confluent.io/rpm/7.1/archive.key\n\
enabled=1\n\
\n\
[Confluent]\n\
name=Confluent repository\n\
baseurl=https://packages.confluent.io/rpm/7.1\n\
gpgcheck=1\n\
gpgkey=https://packages.confluent.io/rpm/7.1/archive.key\n\
enabled=1' >> /etc/yum.repos.d/confluent.repo


RUN rpm --import https://packages.confluent.io/rpm/7.1/archive.key && \
    yum clean all && \
    yum -y install confluent-platform \
      confluent-security

CMD ["/usr/sbin/init"]