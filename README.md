# Fail2ban-Webserver
Setting up fail2ban on a webserver using a cloudflare tunnel.

Fail2ban is a tool for intrusion prevention designed to prevent brutoforce attacks. It monitors logs and block IP addresses that make too many login attempts. Here is a setup of Fail2ban on a apache webserver using a Cloudflare tunnel.

## RemoteIP
Because of the use a Cloudflare tunnel, the visitors are shown in the apache logs as localhost (127.0.0.1). To get the real IP adress you need to activate the remoteip module:

    sudo a2enmod remoteip

Then edit the configuation the website */etc/apache2/sites-enabled/website.conf* (replace with correct configuration file). Add under  <VirtualHost>:

    apacheRemoteIPHeader X-Forwarded-For
    RemoteIPTrustedProxy 127.0.0.1

You may also want to add Cloudflare IP:s as trusted IP adresses. LogFormat in the apache configuration */etc/apache2/apache2.conf* also need to be changed. Find combined LogFormat and change %h to %a:est 

    LogFormat "%a %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined

Test the configuration and reload apache2:

    sudo apache2ctl configtest && sudo systemctl reload apache2

Verify by checking the access logs */var/log/apache2/access.log* (replace with the name of your access log). It should now record the acutal IP adresses.

## Configuring Fail2ban
Install Fail2ban:

    sudo apt install fail2ban

Create a configuration file refering to the correct log files:

    sudo nano /etc/fail2ban/jail.local

Add:

    [DEFAULT]
    bantime = 3600
    findtime = 600 
    maxretry = 3 

    ignoreip = 127.0.0.1/8 ::1

    [apache-auth]
    enabled = true
    port = http,https
    logpath = /var/log/apache2/error.log

    [apache-badbots]
    enabled = true
    port = http,https
    logpath = /var/log/apache2/access.log
    maxretry = 2

    [apache-noscript]
    enabled = true
    port = http,https
    logpath = /var/log/apache2/access.log

    [apache-overflows]
    enabled = true
    port = http,https
    logpath = /var/log/apache2/access.log
    maxretry = 2

Replace error.log and access.log with your website log files.

## Create a WordPress filter
Wordpress is a common target for attackers, even if the site doesn´t use Wordpress it may be useful to set up att wordpress filter:

    sudo nano /etc/fail2ban/filter.d/apache-wordpress.conf

Add:

    [Definition]
    failregex = ^<HOST> .* "(GET|POST) /(wp-login\.php|xmlrpc\.php|wp-admin|wlwmanifest\.xml).* (404|403|400)
    ignoreregex =

Set up a jail configuration:

    sudo nano /etc/fail2ban/jail.d/apache-wordpress.conf

Add:

    [apache-wordpress]
    enabled = true
    filter = apache-wordpress
    logpath = /var/log/apache2/access.log
    maxretry = 3
    findtime = 600
    bantime = 86400

Replace access.log with your website log file.

## Start and verify Fail2ban
Start Fail2ban:

    sudo systemctl enable fail2ban
    sudo systemctl start fail2ban

Check status:

    sudo fail2ban-client status

Check a specific jail:

    sudo fail2ban-client status apache-wordpress

Monitor Fail2ban logs:

    sudo tail -f /var/log/fail2ban.log
s
udo tail -f /var/log/fail2ban.log
