FROM centos:7
MAINTAINER Carl <lpl20150013@163.com>


#====================================解决依赖关系============================================

ENV NGINX_VERSION 1.10.2
ENV PHP_VERSION 7.0.12
ENV LIBICONV_VERSION 1.14

RUN yum -y install bzip* libm* mhash* \
	ImageMagick* php-pear* php-devel* \
	autoconf automake openssh  libiconv* \
	libreoffice-headless  libreoffice-writer lua* \
	libtool cmake make --skip-broken  && \
	yum clean all

#=============================================================================================
#
RUN rpm -ivh http://dl.fedoraproject.org/pub/epel/7Server/x86_64/e/epel-release-7-8.noarch.rpm && \
	yum install -y wget \
	wget zlib-devel zlib* \
	python-setuptools openssh* libmcrypt* \
	freetype-devel libpng-devel libjpeg-devel \
	pcre-devel lrzsz* gcc* p_w_picpath* openoffice* \
	libxslt-devel   freetype-devel  libpng-devel   \
	libcurl-devel openldap* postgresql-devel \
	curl-devel bzip2-devel  libjpeg-devel  readline-devel \
	libmcrypt* glibc* libcurl* \
	screen tree lsof htop iptraf  sysstat inotify-tools  htop\
	mcrypt* libevent* libxml2* \
	post* pcre* gcc-c++ && \
	yum clean all

#====================================安装nginx && php7=========================================
RUN groupadd -r www && \
    useradd -M -s /sbin/nologin -r -g www www


#download nginx libiconv and php7,libiconv1.14版本存在BUG
RUN mkdir -p /usr/local/src/nginx-php && cd $_ && \
	wget -c -O nginx.tar.gz http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz && \
	wget -O php.tar.gz http://php.net/distributions/php-$PHP_VERSION.tar.gz && \
	wget -c -O libiconv.tar.gz http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz && \
	curl -O -SL https://github.com/xdebug/xdebug/archive/XDEBUG_2_4_0RC3.tar.gz


#Make install nginx
RUN cd  /usr/local/src/nginx-php && \
	tar -zxvf nginx.tar.gz && \
	cd nginx-$NGINX_VERSION && \
        ./configure --prefix=/usr/local/nginx \
        --user=www --group=www \
        --error-log-path=/var/log/nginx_error.log \
        --http-log-path=/var/log/nginx_access.log \
        --pid-path=/var/run/nginx.pid \
        --with-pcre \
        --with-http_ssl_module \
        --without-mail_pop3_module \
        --without-mail_imap_module \
        --with-http_gzip_static_module && \
        make && make install

#Make install libiconv,存在BUG可修复
#stdio.in.h找到
#_GL_WARN_ON_USE (gets,"gets is a security hole - use fgets instead");
#注释掉
#添加
##ifdefined(__GLIBC__)&&!defined(__UCLIBC__)&&!__GLIBC_PREREQ(2,16)
# _GL_WARN_ON_USE (gets,"gets is a security hole - use fgets instead");
##endif
RUN cd  /usr/local/src/nginx-php && \
	tar -zxvf libiconv.tar.gz && \
	cd libiconv-$LIBICONV_VERSION && \
	./configure --prefix=/usr/local/libiconv && \
	make -j 4 && make install


#Make install php7
RUN cd  /usr/local/src/nginx-php && \
        tar -zvxf php.tar.gz && \
	cd php-$PHP_VERSION && \
        ./configure --prefix=/usr/local/php \
        --with-config-file-path=/usr/local/php/etc \
        --with-config-file-scan-dir=/usr/local/php/etc/php.d \
        --with-fpm-user=www \
        --with-fpm-group=www \
        --with-mcrypt=/usr/include \
	--with-iconv=/usr/local/libiconv \
        --with-mysqli \
        --with-pdo-mysql \
        --with-openssl \
        --with-gd \
        --with-zlib \
        --with-gettext \
        --with-curl \
        --with-png-dir \
        --with-jpeg-dir \
	--with-freetype-dir \
        --with-xmlrpc \
        --with-mhash \
	--with-openssl \
        --enable-fpm \
        --enable-xml \
        --enable-shmop \
        --enable-sysvsem \
        --enable-inline-optimization \
        --enable-mbregex \
        --enable-mbstring \
        --enable-ftp \
        --enable-gd-native-ttf \
        --enable-mysqlnd \
        --enable-pcntl \
        --enable-sockets \
        --enable-zip \
        --enable-soap \
        --enable-session \
        --enable-opcache \
        --enable-bcmath \
        --enable-exif \
        --enable-fileinfo \
        --disable-rpath \
        --disable-debug \
        --without-pear && \
        make -j 4 && make install


#=======================================================================================================
#
#Add xdebug extension
RUN cd /home/nginx-php && \
    tar -zxvf XDEBUG_2_4_0RC3.tar.gz && \
    cd xdebug-XDEBUG_2_4_0RC3 && \
    /usr/local/php/bin/phpize && \
    ./configure --enable-xdebug --with-php-config=/usr/local/php/bin/php-config && \
    make && \
    cp modules/xdebug.so /usr/local/php/lib/php/extensions/xdebug.so

RUN     cd /home/nginx-php/php-$PHP_VERSION && \
    cp php.ini-production /usr/local/php/etc/php.ini && \
    cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf && \
    cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
#
#=======================================================================================================
#Install supervisor
RUN easy_install supervisor && \
    mkdir -p /var/log/supervisor && \
    mkdir -p /var/run/sshd && \
    mkdir -p /var/run/supervisord

#Add supervisord conf
ADD supervisord.conf /etc/supervisord.conf

#Create web folder
VOLUME ["/data/www", "/usr/local/nginx/conf/ssl", "/usr/local/nginx/conf/vhost", "/usr/local/php/etc/php.d"]
ADD index.php /data/www/index.php

ADD xdebug.ini /usr/local/php/etc/php.d/xdebug.ini

#Update nginx config
ADD nginx.conf /usr/local/nginx/conf/nginx.conf

#========================向容器中添加文件===============================
#Start
ADD start.sh /start.sh
RUN chmod +x /start.sh

#======================指定容器映射到主机的端口=========================
#Set port
EXPOSE 8080 443 9000

#========================================================
#Start it
ENTRYPOINT ["/start.sh"]