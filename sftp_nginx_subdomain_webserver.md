##Nginx users subdomain webserver and SFTP with chrooted, users restricted to their home folder

###Nginx

Nginx working with subdomain based on the users ``\home\USER\WWW`` folders

```Nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        
        server_name ~^(.*)\.YOUR_DOMAIN.COM$;
        root /var/www/html/$1/WWW;
        
        index index.html index.htm;


        location / {
                try_files $uri $uri/ =404;
                autoindex on;
        }

```

###User
Creating user using this bash script

```Bash

#!/bin/bash
if [[ $1 ]]; then
 adduser $1
 usermod -a -G sftpusers $1
 usermod -s /bin/false $1
 gpasswd -a www-data $1
 chmod 0750 /home/$1
 mkdir /home/$1/WWW
 chown -R $1:$1 /home/$1
 nginx -s reload
else
 echo "Type the username"
fi

```

###SSHD Config
To restric the users to their home folder and disable the SSH only allowing access through SFTP.
Edit the ``/etc/ssh/sshd_config`` and added those lines comment the ``Subsystem``. The users in the group ``sudo`` are not matching in this filter.

```Nginx
#Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp

Match Group sftpusers,!sudo
        ChrootDirectory /home
        AllowTCPForwarding no
        X11Forwarding no
        ForceCommand internal-sftp -d %u

```

##References
1. https://en.wikibooks.org/wiki/OpenSSH/Cookbook/SFTP
1. https://www.digitalocean.com/community/questions/automatically-create-users-subdomain-with-sub-folders-in-do-with-nginx
