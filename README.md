# Manually installing a Tileserver

steps from:
http://www.axismaps.com/blog/2012/01/dont-panic-an-absolute-beginners-guide-to-building-a-map-server/
http://wiki.openstreetmap.org/wiki/PostGIS/Installation#Complete_Installation_for_Ubuntu
https://github.com/mapnik/mapnik/wiki/UbuntuInstallation
https://github.com/mapnik/mapnik/wiki/UsingScons

Starting with a fresh Ubuntu 14.04LTS install....

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
Try instead:
```
/etc/init.d/postgresql restart
```
Try to log in to database
```
sudo -u postgres -i
psql gis gisuser
gis=> \d
               List of relations
 Schema |       Name        | Type  |  Owner   
--------+-------------------+-------+----------
 public | geography_columns | view  | postgres
 public | geometry_columns  | view  | postgres
 public | raster_columns    | view  | postgres
 public | raster_overviews  | view  | postgres
 public | spatial_ref_sys   | table | postgres
(5 rows)
```
Ok, time to install mapnik, but first
```
apt-get install software-properties-common
apt-get install -y python-software-properties
add-apt-repository ppa:mapnik/nightly-trunk
```
Now...
```
sudo add-apt-repository ppa:mapnik/nightly-trunk
sudo apt-get update
sudo apt-get install libmapnik libmapnik-dev mapnik-utils python-mapnik
```
Fails on:
```
The following packages have unmet dependencies:
 python-mapnik : Depends: libmapnik (= 3.0.0+dev20150425.git.d89033a-1~trusty1) but 3.0.0+dev20160113.git.6e93b05-1~trusty1 is to be installed
E: Unable to correct problems, you have held broken packages.
```
Remove mapnik:
```
sudo apt-get purge libmapnik* mapnik-* python-mapnik
```
Attempt to build from source instead, so setup build environment:
```
sudo apt-get install libboost-dev libboost-filesystem-dev \
libboost-program-options-dev libboost-python-dev \
libboost-regex-dev libboost-system-dev libboost-thread-dev \

sudo apt-get install \
    libboost-filesystem-dev \
    libboost-program-options-dev \
    libboost-python-dev libboost-regex-dev \
    libboost-system-dev libboost-thread-dev \

sudo apt-get install \
    libicu-dev \
    python-dev libxml2 libxml2-dev \
    libfreetype6 libfreetype6-dev \
    libjpeg-dev \
    libpng-dev \
    libproj-dev \
    libtiff-dev \
    libcairo2 libcairo2-dev python-cairo python-cairo-dev \
    libcairomm-1.0-1 libcairomm-1.0-dev \
    ttf-unifont ttf-dejavu ttf-dejavu-core ttf-dejavu-extra \
    git build-essential python-nose \
    libgdal1-dev python-gdal \
    
wget http://www.freedesktop.org/software/harfbuzz/release/harfbuzz-0.9.26.tar.bz2
tar xf harfbuzz-0.9.26.tar.bz2
cd harfbuzz-0.9.26
./configure && make && sudo make install
sudo ldconfig
cd ../
apt-get update
apt-get upgrade
git clone https://github.com/mapnik/mapnik --depth 10
cd mapnik
git submodule update --init deps/mapbox/variant
./configure
make && sudo make install
```
This of course fails. According to this
https://github.com/mapnik/mapnik/issues/3385
Try using python scons.
```
$ cd mapnik_src_dir
$ python scons/scons.py configure # will configure compilation and save out configuration to a python pickle
$ python scons/scons.py # will compile mapnik sources (running configure first if not done yet)
```
...wait a long time...
...
...
... eventually fails, so don't do this:
```
$ sudo python scons/scons.py install # will install Mapnik (running configure and compiling first if not done yet)
```
instead, let's try to install mapnik 2x from packages.
remove the nightly-trunk ppa:
```
apt-get install ppa-purge
ppa-purge ppa:mapnik/nightly-trunk
```
make sure 2.3 nightly is added to sources:
```
add-apt-repository ppa:mapnik/nightly-2.3
```
and now finally install mapnik 2.3 from pkg:
```
sudo apt-get update
sudo apt-get install libmapnik libmapnik-dev mapnik-utils python-mapnik
```
# also install datasource plugins if you need them
```
sudo apt-get install mapnik-input-plugin-gdal mapnik-input-plugin-ogr\
  mapnik-input-plugin-postgis \
  mapnik-input-plugin-sqlite \
  mapnik-input-plugin-osm
```
test to see if it installed correctly
```
python
>>>import mapnik
>>>
```
great!

