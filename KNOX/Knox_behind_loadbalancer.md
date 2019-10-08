
# LB Knox

## 1. Install apache httpd 
```
yum install httpd
service httpd restart
chkconfig httpd on
```
## 2. Configure httpd

### 2.1. Backup original configuration file 
```
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bkp
```
### 2.2. Edit httpd.conf and make following changes
```
vi /etc/httpd/conf/httpd.conf
```
#### a. Change default port number to 8443, default is set to 80 (Listen 80)
```
Listen 8443
```
#### b. Comment out ServerAdmin email address
```
#ServerAdmin root@localhost
```

#### c. Un-comment/Add following module entries
```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
LoadModule lbmethod_bytraffic_module modules/mod_lbmethod_bytraffic.so
LoadModule lbmethod_bybusyness_module modules/mod_lbmethod_bybusyness.so
```

#### d. At the end of the httpd.conf file, add the following line to read the custom configuration file:
```
include conf/knox-cluster.conf
```

## 3. Create a custom conf file for ranger in httpd conf directory.
```
vi /etc/httpd/conf/knox-cluster.conf
```

Add the following lines (Make sure <VirtualHost *:8443> port number should match with the default port we set in the httpd.conf file in the previous step)

```
#
# This is the Apache server configuration file providing SSL support.
# It contains the configuration directives to instruct the server how to
# serve pages over an https connection. For detailing information about these
# directives see <URL:http://httpd.apache.org/docs/2.2/mod/mod_ssl.html>
#
# Do NOT simply read the instructions in here without understanding
# what they do.  They're here only as hints or reminders.  If you are unsure
# consult the online docs. You have been warned.

#Listen 80
<VirtualHost *:8443>
        ProxyRequests off
        ProxyPreserveHost on

        Header add Set-Cookie "ROUTEID=.%{BALANCER_WORKER_ROUTE}e; path=/" env=BALANCER_ROUTE_CHANGED

        <Proxy balancer://rangercluster>
                BalancerMember http://knox1.hortonworks.com:8443 loadfactor=1 route=1
                BalancerMember http://knox2.hortonworks.com:8443 loadfactor=1 route=2

                Order Deny,Allow
                Deny from none
                Allow from all

                ProxySet lbmethod=byrequests scolonpathdelim=On stickysession=ROUTEID maxattempts=1 failonstatus=500,501,502,503 nofailover=Off
        </Proxy>

        # balancer-manager
        # This tool is built into the mod_proxy_balancer
        # module and will allow you to do some simple
        # modifications to the balanced group via a gui
        # web interface.
        <Location /balancer-manager>
                SetHandler balancer-manager
                Order deny,allow
                Allow from all
        </Location>

       ProxyPass /balancer-manager !
       ProxyPass / balancer://knoxcluster/
       ProxyPassReverse / balancer://knoxcluster/

</VirtualHost>
```

## 4. Restart httpd
```
service httpd restart
```

## 5. Access webhdfs using LB Knox url
```
curl -ikv -u gulshad:Welcome123 http://c1230-node4.squadron.support.hortonworks.com:8443/gateway/default/webhdfs/v1/user?op=LISTSTATUS
```

