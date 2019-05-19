# Install & Enable SSL on Apache httpd server

## Part I: Install httpd webserver

### Install httpd package
```
# yum install httpd
```

### Start httpd server
```
# systemctl start httpd 
# systemctl enable httpd 
```
### Create a php page to verify if its working as expected
```
vi /etc/phpldapadmin/config.php
```

```
<html>
<body>
<div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
<?php
   echo "<br> Welcome to DABSTER <br> <br>";
   print Date("Y/m/d");
?>
</div>
</body>
</html>
```

### Access the test web page from client PC with web browser. Following page is shown.

![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/httpd-test-webpage.png)


## Part II: Setup SSL on Apache httpd server

### Install required packages
```
# yum install httpd -y
# yum install mod_ssl -y
```

### Create ssl certificate and private key
```
# openssl req -new -newkey rsa:2048 -x509 -sha256 -days 365 -nodes -out /etc/pki/tls/certs/localhost.crt -keyout /etc/pki/tls/private/localhost.key
```

### Configure httpd for ssl
```
# vi /etc/httpd/conf.d/ssl.conf
```
_Update ServerName, SSLCertificateFile & SSLCertificateKeyFile_
```
<VirtualHost *:443>
    ServerName webserver.dabsterinc.com
    SSLEngine on
    SSLCertificateFile "/etc/pki/tls/certs/localhost.crt"
    SSLCertificateKeyFile "/etc/pki/tls/private/localhost.key"
</VirtualHost>
```
### Restart Apache webserver
```
# service httpd restart
```

### Create some dummy files for testing purpose
```
# mkdir /var/www/html/test;touch /var/www/html/test/{1..10}
```

### Open your webbrowser with https
https://webserver.dabsterinc.com/test

