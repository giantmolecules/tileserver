# Manually installing a Tileserver

Start with a fresh Ubuntu 14.04LTS install

Install LAMP: https://www.linode.com/docs/websites/lamp/lamp-on-ubuntu-14-04
```
restore /var/www
restore apache2.conf
restore sites-available
```
Disable default site:
```
a2dissite 000-default.conf
```
Install mod_python for apache2:
```
apt-get install libapache2-mod-python (referenced in maps.generalradio.org.conf)
```
Install MySQL:
```
sudo apt-get install mysql-server 
mysql_secure_installation
```
Install PHP:
```
sudo apt-get install php5 php-pear
sudo apt-get install php5-mysql
```
Set (uncomment) the following in /etc/php5/apache2/php.ini
```
error_reporting = E_COMPILE_ERROR|E_RECOVERABLE_ERROR|E_ERROR|E_CORE_ERROR
error_log = /var/log/php/error.log
max_input_time = 30
```
Create the log directory for PHP and give the Apache user ownership:
```
sudo mkdir /var/log/php
sudo chown www-data /var/log/php
```
Reload Apache:
```
sudo service apache2 reload
```
- [ ]set FQDN in apache2.conf?



