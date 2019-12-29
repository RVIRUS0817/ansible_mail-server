# ansible_mail-server

![スクリーンショット 2019-12-02 20 01 17](https://user-images.githubusercontent.com/5633085/69954431-9f1f3680-153e-11ea-8dc9-f8dc55aead08.jpg)


## Environment
- CentOS7
- postfix
- dovecot
- postfixadmin 3.2
- php7.0
- H2O 2.2.5
- MySQL5.7
- vhost directory
```
# ls /var/vmail/
bin  deleted-vhosts  vhosts
```
- DocumentRoot
```
/var/www/postfixadmin
```

## 1. setting user


It is assumed that you have set ssh's config with the initial user (root)
In ansible you can specify your favorite user and enable it with sudo no pass so please change the following hoge and add the public key.

- hosts
- mail.yml
- roles/common/tasks/user.yml

## 2. setting H2O

The web server uses H2O.
httpsization uses letsencrypt.

- roles/h2o/files/h2o.conf
- roles/h2o/files/conf.d/hoge.conf
- roles/h2o/tasks/main.yml

Please change hoge to domain name!! you have to setting basic.

## 3. make dh2048.pem,dh512.pem

Create dh2048.pem and dh512.pem on the server. Copy this to `roles/ssl/files` .

```
$ sudo mkdir -p /etc/postfix/ssl/dhparams/
$ sudo openssl dhparam -out /etc/postfix/ssl/dhparams/dh2048.pem 2048
$ sudo openssl dhparam -out /etc/postfix/ssl/dhparams/dh512.pem 512
$ sudo mkdir -p /etc/postfix/ssl/selfsigned/
$ sudo openssl req -new -newkey rsa:4096 -days 3658 -sha256 -nodes -x509 \
      -subj "/C=JP/ST=Shizuoka/L=Shizuoka/O=Mailserver certificate/OU=Mail/hoge.jp/emailAddress=admin@hoge.jp" \
      -keyout /etc/postfix/ssl/selfsigned/privkey.pem \
      -out /etc/postfix/ssl/selfsigned/cert.pem
```

## 4. setting dbpass domain
- group_vars/all

## 5. dry-run

````
$ ansible-playbook -i hosts mail.yml -C
````

## 6. run

````
$ ansible-playbook -i hosts mail.yml
````

## 7. setting MySQL

Set up MySQL manually. Let's set up a dedicated DB and authority for wordpress.

・MySQL

```
$ grep 'temporary password' /var/log/mysqld.log
$ mysql_secure_installation
change root pass
all Y
```

・create podtfix DB and user,pass

```
$ mysql -u root -p
> create database postfix;
> grant all privileges on postfix.* to postfix@localhost identified by 'xxxxxxxxxxx';
```

## 8. setting letsencrypt

Let's get a certificate with letsencrypt when wordpress can be viewed with http.
Please change hoge to domain name!!

```
# vim /etc/h2o/conf.d/hoge.jp
#"hoge.jp:443":
Comment out all below
```

```
# systemctl restart h2o
```

```
# /etc/letsencrypt/letsencrypt-auto certonly --standalone -d hoge.jp --agree-tos

```

## 9. access postfixadmin

https://admin.hoge.jp/postfixadmin

- change pass and add `roles/postfixadmin/files/postfixadmin/config.inc.php`
```
CONF['setup_password'] = 'xxxxxxxxx';
```

- add DNS record
```
mail.hoge.jp A xxx.xxx.xxx.xxx
admin.hoge.jp A xxx.xxx.xxx.xxx
hoge.jp TXT v=spf1 +a:mail.hoge.jp ~all
hoge.jp MX xxx.xxx.xxx.xxx
```

Enjoy!!!🤣
