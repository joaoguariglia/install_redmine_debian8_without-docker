# install_redmine_debian8_without-docker
Installing redmine without docker, in the normal way on Debian8 netinst version, as an XLC or virtualhost.


# adduser suporte
# apt-get install sudo
# sudo adduser suporte sudo

$ sudo aptitude install mysql-server mysql-client libmysqlclient-dev gcc build-essential zlib1g zlib1g-dev zlibc ruby-zip libssl-dev libyaml-dev libcurl4-openssl-dev ruby gem libapache2-mod-passenger apache2-mpm-prefork apache2-dev libapr1-dev libxslt1-dev checkinstall libxml2-dev ruby-dev vim libmagickwand-dev imagemagick

$ cd /opt/
$ sudo mkdir redmine
$ sudo chown -R suporte redmine
$ cd redmine
$ wget $redmine.tar.gz
$ tar xzf $redmine.tar.gz
$ cd redmine-3.4.2

$ mysql --user=root --password=password

CREATE DATABASE redmine CHARACTER SET utf8;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
exit

$ cp config/database.yml.example config/database.yml
$ nano config/database.yml

## change password to password and username to redmine
production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: password

$ sudo gem install bundler
$ bundle install --without development test
$ bundle exec rake generate_secret_token
$ RAILS_ENV=production bundle exec rake db:migrate
$ RAILS_ENV=production bundle exec rake redmine:load_default_data
$ bundle exec ruby bin/rails server -b 10.0.10.228 webrick -e production
Web Test: http://10.0.10.228:3000

$ sudo chown -R www-data files log tmp public/plugin_assets
$ sudo chmod -R 755 files log tmp public/plugin_assets
$ sudo chown www-data:www-data Gemfile.lock
$ sudo ln -s /opt/redmine/redmine-3.4.2/public/ /var/www/html/redmine
$ sudo nano /etc/apache2/sites-available/master.conf
## create this new file

<VirtualHost *:80>

ServerAdmin admin@example.com
Servername localhost
DocumentRoot /var/www/html/

        <Location /redmine>
                RailsEnv production
                RackBaseURI /redmine
                Options -MultiViews
        </Location>

</VirtualHost>

$ sudo a2dissite 000-default.conf
$ sudo a2ensite master.conf
$ nano /etc/apache2/mods-available/passenger.conf 
## add [PassengerUser www-data] in the file below

<IfModule mod_passenger.c>
  PassengerRoot /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini
  PassengerDefaultRuby /usr/bin/ruby
  PassengerUser www-data
</IfModule>

$ sudo service apache2 restart
## now access http://10.0.10.228/redmine

Obs: initial user
admin/admin



