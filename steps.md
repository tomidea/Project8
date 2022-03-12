## Configure Apache As A Load Balancer
1. Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb
 
 <img width="1028" alt="apache instance" src="https://user-images.githubusercontent.com/51254648/158013110-9ee58a87-35a0-4e91-a042-254c87da44e7.png">


2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

3. Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

#Install apache2

- sudo apt update
- sudo apt install apache2 -y
- sudo apt-get install libxml2-dev*

#Enable following modules:

- sudo a2enmod rewrite
- sudo a2enmod proxy
- sudo a2enmod proxy_balancer
- sudo a2enmod proxy_http
- sudo a2enmod headers
- sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
- sudo systemctl restart apache2

<img width="639" alt="apache2 running" src="https://user-images.githubusercontent.com/51254648/158013112-b44c60fc-44b1-4a49-ab31-9316a2dbffd9.png">


#### Configure load balancing

- sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>
   
               <Proxy "balancer://mycluster'>
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
                BalancerMember http://<WebServer3-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>*

       *ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/*

<img width="632" alt="Configure LB" src="https://user-images.githubusercontent.com/51254648/158013115-d718adf1-6435-4ef0-9203-793789697bce.png">


#Restart apache server

- sudo systemctl restart apache2
  
  - Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php

   <img width="1222" alt="verify config" src="https://user-images.githubusercontent.com/51254648/158013118-3aee27bc-037e-40bf-9814-67480bbeb34c.png">

 
  - unmount /var/log/httpd/ from your Web Servers to the NFS server and make sure that each Web Server has its own log directory.
      *sudo umount -f /var/log/httpd*
  
  - Open two ssh/Putty consoles for both Web Servers and run following command:
      *sudo tail -f /var/log/httpd/access_log
  
  Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.

   <img width="820" alt="web 1" src="https://user-images.githubusercontent.com/51254648/158013120-60a43010-0ffa-44f7-8d8f-9283ba294f6d.png">

 <img width="823" alt="web 2" src="https://user-images.githubusercontent.com/51254648/158013122-c314f7ff-1ed2-41d3-bb21-328ae9a4f411.png">

  
  ## Configure Local DNS Names Resolution
  #Open this file on your LB server

      - sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
 
   <img width="422" alt="local dns" src="https://user-images.githubusercontent.com/51254648/158013124-94d64289-0ff8-4460-ac5f-94df42747ab2.png">

  
  - Update your LB config file with those names instead of IP addresses.
  BalancerMember http://Web1:80 loadfactor=5 timeout=1
  BalancerMember http://Web2:80 loadfactor=5 timeout=1

   <img width="506" alt="update config file" src="https://user-images.githubusercontent.com/51254648/158013126-8b605447-57ce-402b-8856-5d1ea641fd2b.png">

  
  curl your Web Servers from LB locally curl http://Web1 or curl http://Web2

<img width="605" alt="curl web 1" src="https://user-images.githubusercontent.com/51254648/158013127-8800e595-779d-4a35-9cc5-91bfb617145e.png">
<img width="586" alt="curl web 2" src="https://user-images.githubusercontent.com/51254648/158013128-0e8af26c-cca4-41b3-9041-9ca2a17fb782.png">

