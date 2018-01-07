FROM ubuntu:16.04

LABEL maintainer="Krystian Jasnos <dzejson91@gmail.com>"

# Install base
RUN apt-get update \
	&& apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4F4EA0AAE5267A6C \
	&& apt-get install -y -q --no-install-recommends \
	software-properties-common python-software-properties \
	supervisor curl language-pack-en

RUN sed -i 's/^\(\[supervisord\]\)$/\1\nnodaemon=true/' /etc/supervisor/supervisord.conf \
	&& adduser user \
	&& echo "root:root" | chpasswd \
	&& echo "user:pass" | chpasswd

# Install SSH
RUN apt-get install -y -q --no-install-recommends \
	openssh-server ssh \
	&& mkdir /var/run/sshd \
	&& /usr/bin/ssh-keygen -A \
	&& sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
COPY ./supervisor/ssh.conf /etc/supervisor/conf.d/

# Install PHP5.6 & PHP7.2
RUN LC_ALL=en_US.UTF-8 add-apt-repository -y ppa:ondrej/php \
	&& apt-get update \
	&& apt-get install -y -q --no-install-recommends \
		php7.2 libapache2-mod-php7.2 php7.2-fpm php7.2-mysql php7.2-xml php7.2-curl php7.2-mbstring php7.2-cli php7.2-common php7.2-gd php7.2-imap php7.2-intl php7.2-json php7.2-pspell php7.2-readline php7.2-zip php7.2-bz2 \
		php5.6 libapache2-mod-php5.6 php5.6-fpm php5.6-mysql php5.6-xml php5.6-curl php5.6-mbstring php5.6-cli php5.6-common php5.6-gd php5.6-imap php5.6-intl php5.6-json php5.6-pspell php5.6-readline php5.6-zip php5.6-bz2 \
	&& phpenmod -v ALL -s ALL json opcache gd pdo zip intl pdo_mysql
RUN mkdir -p /var/run/php
COPY ./supervisor/php-fpm.conf /etc/supervisor/conf.d/

# Install Node & NPM
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - \
	&& apt-get install -y nodejs 
	
# Intall Apache2
RUN apt-get install -y -q --no-install-recommends \ 
	apache2 libapache2-mod-fastcgi \
	&& a2enmod alias headers expires rewrite ssl actions fastcgi \
	&& mkdir -p /etc/apache2/ssl \
	&& mkdir -p /www ; chown -R www-data:www-data /www
COPY ./supervisor/apache2.conf /etc/supervisor/conf.d/
COPY ./apache2 /etc/apache2
COPY ./www/* /www/

# Install other apps
RUN apt-get install -y -q --no-install-recommends \
	cron git nano htop composer
COPY ./supervisor/cron.conf /etc/supervisor/conf.d/
COPY ./cron/* /etc/cron.d/conf.d/

# Install shortcuts to root & user
COPY ./user/.shortcuts /.shortcuts
RUN chmod 775 /.shortcuts \
	&& echo "source /.shortcuts" >> /home/user/.profile \
	&& echo "source /.shortcuts" >> /root/.profile
	
# Clean
RUN apt-get clean && rm -r /var/lib/apt/lists/*

WORKDIR /www
VOLUME /www

#ENTRYPOINT /bin/sh -c
CMD ["/usr/bin/supervisord"]
