FROM ubuntu:16.04

# Update & install base
RUN apt-get update
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4F4EA0AAE5267A6C
RUN apt-get install -y -q --no-install-recommends \
	language-pack-en software-properties-common python-software-properties \
	supervisor curl cron git nano htop sudo
	

# Install PHP5.6 & PHP7.2
RUN LC_ALL=en_US.UTF-8 add-apt-repository -y ppa:ondrej/php && apt-get update

RUN apt-get install -y -q --no-install-recommends \ 
	php7.2 libapache2-mod-php7.2 php7.2-fpm php7.2-mysql php7.2-xml php7.2-curl php7.2-mbstring php7.2-cli php7.2-common php7.2-gd php7.2-imap php7.2-intl php7.2-json php7.2-pspell php7.2-readline php7.2-zip php7.2-bz2

RUN apt-get install -y -q --no-install-recommends \
	php5.6 libapache2-mod-php5.6 php5.6-fpm php5.6-mysql php5.6-xml php5.6-curl php5.6-mbstring php5.6-cli php5.6-common php5.6-gd php5.6-imap php5.6-intl php5.6-json php5.6-pspell php5.6-readline php5.6-zip php5.6-bz2 \
	composer


# Intall Apache2
RUN apt-get install -y -q --no-install-recommends \ 
	apache2 libapache2-mod-fastcgi \
	&& a2enmod alias headers expires rewrite ssl actions fastcgi

# Install SSH
RUN apt-get install -y -q --no-install-recommends \
	openssh-server ssh sshfs \
	&& mkdir /var/run/sshd \
	&& /usr/bin/ssh-keygen -A \
	&& sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config


# Install Node & NPM
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - \
	&& apt-get install -y nodejs


# Clean files
RUN apt-get clean && rm -rf /var/lib/apt/lists/*


# Configure supervisor & cron
RUN sed -i 's/^\(\[supervisord\]\)$/\1\nnodaemon=true/' /etc/supervisor/supervisord.conf
COPY ./supervisor/* /etc/supervisor/conf.d/
COPY ./cron/* /etc/cron.d/conf.d/


# Configure Apache2 & PHP
RUN rm -rf /etc/apache2/sites-enabled/* \
	&& mkdir -p /etc/apache2/ssl /www
COPY ./apache2 /etc/apache2

# Configure PHPs
RUN phpenmod -v ALL -s ALL json opcache gd pdo zip intl pdo_mysql \
	&& mkdir -p /var/run/php

VOLUME /www
WORKDIR /www

# Add User & shortcuts
RUN useradd -g 33 -G sudo user
RUN echo "user:user"|chpasswd && echo "root:root" | chpasswd
COPY ./user/.shortcuts /.shortcuts
RUN echo "source /.shortcuts" >> /root/.profile
#RUN echo "source /.shortcuts" >> /home/user/.profile
	
RUN chown www-data:www-data /www && chmod 775 /www


#ENTRYPOINT /bin/sh -c
CMD ["/usr/bin/supervisord"]
