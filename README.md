# Reinproject.org

The code of reinproject.org.

## How to set up the server

Given a Ubuntu 16.04 droplet, you need to run the following commands to get Rein set up.
First, set 2 variables for the repo address and for the work directory. You can export them to env.

```
REPO="https://github.com/adsalpha/reinproject.org"
WORKDIR="/opt/reinproject.org"
```

Next, install all the necessary packages and clone this Git repo.

```
sudo apt update
sudo apt install -y python-pip nginx letsencrypt
sudo pip install uwsgi flask
sudo mkdir $WORKDIR $WORKDIR/ssl
sudo git clone $REPO $WORKDIR
```

For SSL we use Let's Encrypt. Issue a certificate for (www.)reinproject.org.

```
letsencrypt certonly --webroot -w /var/www/html -d reinproject.org -d www.reinproject.org
```

Let's Encrypt certificates are valid for 90 days only. That's why you need to set up a `cron` job to renew them. Execute `sudo crontab -e`, choose your favorite text editor and append the following string to the end of the file:
```
0 0 * * 1 /usr/bin/letsencrypt renew >> /var/log/letsencrypt-renew.log
```

This repo supplies the configuration files you need for Nginx, Systemd, or Upstart. Symlink them to local directories.

```
sudo rm /etc/nginx/sites-available/default
sudo ln -s $WORKDIR/misc/nginx/reinproject.org /etc/nginx/sites-available/reinproject.org
sudo ln -s $WORKDIR/misc/nginx/reinproject.org /etc/nginx/sites-enabled/reinproject.org
sudo ln -s $WORKDIR/misc/systemd/reinproject.org.service /lib/systemd/system/reinproject.org.service
sudo ln -s /etc/letsencrypt/live/reinproject.org/fullchain.pem $WORKDIR/ssl/fullchain.pem
sudo ln -s /etc/letsencrypt/live/reinproject.org/privkey.pem $WORKDIR/ssl/privkey.pem
```

Reboot the server, then start the Systemd service. You need to do this manually after every reboot.

```
systemctl start reinproject.org.service
```

## How to add language support

Language support is designed to use the first two letters of the 'Accept-Language' HTML header. Thus, if a user's OS is in English, reinproject.org will use 'en' as the language setting. For Russian, it's 'ru', for Romanian - 'ro', for French - 'fr'.

To add language support, add a directory with the two language letters as its name to app/templates.
