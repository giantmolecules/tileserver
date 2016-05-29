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
Left to do:

- [ ] Set FQDN in apache2.conf?
- [ ] Setup Firewall using IPTABLES

Install PostGIS. See: http://wiki.openstreetmap.org/wiki/PostGIS/Installation#Complete_Installation_for_Ubuntu
```
sudo apt-get install postgresql postgresql-contrib postgis
```
Create DB *gis* and user *gisuser*:
```
sudo -u postgres createuser gisuser
sudo -u postgres createdb --encoding=UTF8 --owner=gisuser gis
```
Enable PostGIS on database:
```
sudo -u postgres -i
psql --username=postgres --dbname=gis -c "CREATE EXTENSION postgis;"
```
this fails because postgis-scripts isn't installed, do the following:
```
apt-get install postgis*
```
now succeeds with CREATE EXTENSION
```
psql --username=postgres --dbname=gis -c "CREATE EXTENSION postgis_topology;"
CREATE EXTENSION
exit
```
Change ident and md5 to trust in /etc/postgresql/9.3/main/pg_hba.conf
```
local   all         all                               ident
host    all         all         127.0.0.1/32          md5
```
Reload database:
```
sudo -u postgres /usr/lib/postgresql/9.3/bin/pg_ctl reload
```
Yields:
```
pg_ctl: no database directory specified and environment variable PGDATA unset
Try "pg_ctl --help" for more information.
```

