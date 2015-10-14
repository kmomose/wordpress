FROM centos:centos6
RUN /bin/cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
RUN yum -y install httpd php php-mysql mysql-server tar wget php-gd
WORKDIR /tmp/
RUN wget https://ja.wordpress.org/latest-ja.tar.gz
RUN tar xvfz ./latest-ja.tar.gz
RUN rm -f ./latest-ja.tar.gz
RUN mv wordpress/* /var/www/html/
RUN cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
RUN sed -i -e 's/database_name_here/wordpress/g' -e 's/username_here/wordpress/g' -e 's/password_here/wppass/g' /var/www/html/wp-config.php
RUN sed -i -e 's/AllowOverride\ None/AllowOverride\ AuthConfig/g' /etc/httpd/conf/httpd.conf
RUN chown -R apache.apache /var/www/html/
RUN service mysqld start && mysql -u root -e "CREATE DATABASE wordpress; GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost' IDENTIFIED BY 'wppass'; FLUSH PRIVILEGES;" &&  service mysqld stop
RUN echo -e "service mysqld start\nservice httpd start\n/bin/bash" > /startService.sh
RUN chmod o+x /startService.sh
EXPOSE 80
CMD /startService.sh
