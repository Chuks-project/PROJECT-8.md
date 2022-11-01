## LOAD BALANCER SOLUTION WITH APACHE

In our set up in Project-7 we had 3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server

To solve this problem, we need a single point of entry which routes traffic accross our webserver(3). Depending our configuration, the webserver can equally share the incoming request.In order to hide all this complexity and to have a single point of access with a single public IP address/name, a Load Balancer can be used. A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way. We will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

In this project, We will adopt the Horizontal Scalability which is one of the approaches used to cater for increased trafic. Horizontal scaling allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. Adjustment of number of servers can be done manually or automatically (for example, based on some monitored metrics like CPU and Memory load).

## PREREQUISITES
Ensure you have the following servers installed and configured within Project-7 running:

1. Two RHEL8 Web Servers
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server


- Configure Apache As A Load Balancer
- Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb,

- Open TCP port 80 on Project-8-apache-lb

- Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

`   
 `   #Install apache2`
     sudo apt update
     sudo apt install apache2 -y
 `    sudo apt-get install libxml2-dev

     #Enable following modules:
     sudo a2enmod rewrite
     sudo a2enmod proxy
     sudo a2enmod proxy_balancer
     sudo a2enmod proxy_http
     sudo a2enmod headers
     sudo a2enmod lbmethod_bytraffic
`     

 - #Restart apache2 service
    
    `sudo systemctl restart apache2`
    
 - Make sure apache2 is up and running
    
    `sudo systemctl status apache2`



 - sudo vi /etc/apache2/sites-available/000-default.conf

 - #Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

`
   <Proxy "balancer://mycluster">
             BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
             BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
             ProxySet lbmethod=bytraffic
  `           # ProxySet lbmethod=byrequests
     </Proxy>

      ProxyPreserveHost On
      ProxyPass / balancer://mycluster/
      ProxyPassReverse / balancer://mycluster/
 `     `   

- #Restart apache server

      `sudo systemctl restart apache2`
      
      
- Bytraffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.

- You can also study and try other methods, like: bybusyness, byrequests, heartbeat

- Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
      
   `http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php                                                                     `
   
- If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

- Open two ssh/Putty consoles for both Web Servers and run following command:

  ` sudo tail -f /var/log/httpd/access_log                                                                                `
  
- Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.

- You will also notice that while refreshing you browser, the traffic will be distributed or shared between your two webservres as seen below:

![162](https://user-images.githubusercontent.com/65022146/199266352-c3fc9017-a4c0-486a-9d5a-2a57a10df882.png)
![206](https://user-images.githubusercontent.com/65022146/199266387-15c8e3f1-f346-46c0-b952-d9f284333d0b.png)
