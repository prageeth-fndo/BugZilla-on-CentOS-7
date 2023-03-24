# BugZilla-on-CentOS-7
Guide to install Bugzilla 5.0.3 on CentOS 7.

This guide will help you to properly install Bugzilla 5.0.6 on CentOS 7 distribution with MySQL 5.7, Perl and Apache 2.6. <br><br>
In this guideline, SOURCE INSTALLATION was done for the MySQL 5.7. <br>

### INITIAL SETUP
```
Yum update -y
```
### MYSQL INSTALLATION AND CONFIGURATION
```
yum groupinstall "Development Tools" -y
yum install cmake ncurses-devel -y
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.20.tar.gz
tar -xzf mysql-boost-5.7.20.tar.gz
cd mysql-5.7.20/
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr/local/mysql -DWITH_BOOST=/root/mysql-5.7.20/boost/
make
make install
cd /usr/local/mysql/
vim my.cnf
useradd mysql -s /sbin/nologin
chown -R mysql.mysql /usr/local/mysql/
bin/mysqld --initialize --user=mysql
bin/mysqld_safe --user=mysql & 
bin/mysql -u root -p -S /usr/local/mysql/mysql.sock
ALTER USER ' root'@'localhost' IDENTIFIED BY 'root';
cp /root/mysql-5.7.20/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
ps aux | grep mysql
kill -9 PIDs
service mysqld start
service mysqld status
```
> change my.cnf path to /tmp/mysql.sock
```
chkconfig --level 345 mysqld on
export PATH=$PATH:/usr/local/mysql/bin
```
### DATABASE CREATION
```
mysql -u root -p
CREATE DATABASE bugzilla CHARACTER SET utf8mb4;
CREATE USER 'bugzilla'@'localhost' IDENTIFIED BY 'bugzilla@123';
GRANT ALL PRIVILEGES ON bugzilla.* TO 'bugzilla'@'localhost';
```
### BUGZILLA INSTALLATION
```
yum install mod_ssl php-mysql gcc perl* mod_perl-devel -y
yum install -y epel-release
yum install -y mod_perl
yum install iptables-services -y
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
service iptables save
/sbin/chkconfig httpd on
wget https://ftp.mozilla.org/pub/mozilla.org/webtools/bugzilla-5.0.6.tar.gz
tar zxvf bugzilla-5.0.3.tar.gz
mv bugzilla-5.0.3 /var/www/html/
cd /var/www/html/
mv bugzilla-5.0.3/ bugzilla
cd bugzilla/
./checksetup.pl --check-modules
perl install-module.pl --all 
./checksetup.pl
vi localconfig
```
> change localconfig fie attributes as follows
```
$db_host = '127.0.0.1'
$db_name = 'bugzilla'
$db_user = 'bugzilla'
$db_pass = 'bugzilla@123' 
$db_sock = '/tmp/mysql.sock'
```
```
./checksetup.pl
```
### APACHE CONFIGURATION
```
yum install httpd
systemctl enable httpd
vi /etc/httpd/conf.d/bugzilla.conf
```
> copy and paste following segement
```
<VirtualHost *:80>
ServerAdmin admin@example.com
DocumentRoot /var/www/html/bugzilla/
ServerName bugzilla.example.com
ServerAlias www.bugzilla.example.com
<Directory /var/www/html/bugzilla/>
AddHandler cgi-script .cgi
Options +Indexes +ExecCGI
DirectoryIndex index.cgi
AllowOverride Limit FileInfo Indexes Options AuthConfig
</Directory>
ErrorLog /var/log/httpd/bugzilla.example.com-error_log
CustomLog /var/log/httpd/bugzilla.example.com-access_log common
</VirtualHost>
```
```
chown -R apache:apache /var/www/html/bugzilla
systemctl stop firewalld
systemctl disable firewalld
nano /etc/sysconfig/selinux 
```
> set SELINUX=disabled
```
reboot
```
### FIREWALL PERMISSION
```
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --zone=public --add-port=443/tcp --permanent
sudo firewall-cmd --reload
```
> Or you can disable the firewall
```
systemctl disable firewalld
```
### OLD DB RESTORING
> Drop the existing bugzilla database and create an empty bugzilla database.
```
mysql -u root -p bugzilla < /root/qabugsbackup.sql
```
