# Install & test apache httpd

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
