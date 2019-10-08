# Setup High Availability (HA) for Apache Ranger using apache httpd webserver

Configiration and versions used:

```
Load balancer node	:	lb.hortonworks.com
Ranger (old)		:	hdp1.hortonworks.com
Ranger (new)		:	hdp2.hortonworks.com


HDP Version 	  	:	HDP-2.5
Ambari Version		:	Ambari-2.5
Linux			:	Centos 7
```


### 1.	Login to the node where you want to setup load-balancer.
ssh lb.hadoop.com

2.	Install https and its dependencies (apr and apr-util)
```
[root@lb ~]# yum install httpd
```

3.	Backup original configuration file
```
[root@lb ~]# cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bkp
```

4.	Edit the httpd.conf  file and make the following changes
```
[root@lb ~]# vi /etc/httpd/conf/httpd.conf
```

a.	Change default port number to 6080, default is set to 80 (Listen 80)
```
Listen 8443
```

b.	Un-comment/Add following module entries
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

c.	Comment out ServerAdmin email address
```
#ServerAdmin root@localhost
```

d.	At the end of the httpd.conf file, add the following line to read the custom configuration file:
```
include conf/ranger-cluster.conf
```

5.	Create a custom conf file for ranger in httpd conf directory.
```
[root@lb ~]#  vi /etc/httpd/conf/ranger-cluster.conf
```

Add the following lines (Make sure <VirtualHost *:6080> port number should match with the default port we set in the httpd.conf file in the previous step)
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
<VirtualHost *:6080>
        ProxyRequests off
        ProxyPreserveHost on

        Header add Set-Cookie "ROUTEID=.%{BALANCER_WORKER_ROUTE}e; path=/" env=BALANCER_ROUTE_CHANGED

        <Proxy balancer://rangercluster>
                BalancerMember http://hdp1.hadoop.com:6080 loadfactor=1 route=1
                BalancerMember http://hdp2.hadoop.com:6080 loadfactor=1 route=2

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
       ProxyPass / balancer://rangercluster/
       ProxyPassReverse / balancer://rangercluster/

</VirtualHost>
```

6.	Restart httpd
```
[root@lb ~]# service httpd restart
```

7.	Start service on boot-up
```
[root@lb ~]# chkconfig httpd on
```

# From Ambari
1.	Select Ranger service click Service Actions > Enable Ranger Admin HA

2.	On the Get Started page, enter the load-balancer URL and port number (in this example, http://lb.hortonworks.com:6080), then click Next.

3.	On the Select Hosts page, confirm the host assignments, then click Next.

4.	Check the settings on the Review page, then click Next.

5.	Click Complete to complete the installation

6.	Restart All Required Services

7.	Test the load-balancer and Ranger HA configuration. Select Ranger > Service Actions > Stop on one of the Ranger hosts. 

8.	Open Ranger UI using load-balancer IP and Port

