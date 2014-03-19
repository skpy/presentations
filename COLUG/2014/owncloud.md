## Intro
[ownCloud](http://owncloud.org/) "provides universal access to your files via the web, your computer or your mobile devices — wherever you are."

What follows are the bare minimum steps required to get a functional ownCloud installation on CentOS 6.5 and Ubuntu 13.10.

These examples do not promote performance or security.  You have been warned.

## ownCloud on CentOS
```shell
sudo yum -y update
sudo yum -y install sqlite httpd php php-gd php-xml php-mbstring php-intl php-pdo
cd /var/www/html
sudo tar jxf ~/owncloud-6.0.2.tar.bz2
sudo chown -R apache owncloud
sudo cat << EOF >> /etc/httpd/conf.d/owncloud.conf:
<Directory /var/www/html/owncloud>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    allow from all
</Directory>
EOF
```

## ownCloud on Ubuntu
```shell
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install apache2 libapache2-mod-php5 php5-gd php5-json php5-sqlite php5-curl php5-intl php5-mcrypt
sudo a2enmod rewrite
sudo service apache2 restart
sudo tar jxf ~/owncloud-6.0.2.tar.bz2
sudo chown -R www-data owncloud
```

## Other Tools
* [ownCloud Sync clients](http://owncloud.org/sync-clients/)
* [ownCloud Android Client](https://play.google.com/store/apps/details?id=com.owncloud.android)
* [CalDAV Sync for Android](https://play.google.com/store/apps/details?id=org.dmfs.caldav.lib)
* [CardDAV Sync for Android](https://play.google.com/store/apps/details?id=org.dmfs.carddav.Sync)
