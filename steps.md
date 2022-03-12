## Configure Apache As A Load Balancer
Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb
img

Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

*#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

*#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

*#Restart apache2 service
sudo systemctl restart apache2
img

#### Configure load balancing

sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
                BalancerMember http://<WebServer3-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
img

#Restart apache server

sudo systemctl restart apache2
  
  - Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php
img
 
  - unmount /var/log/httpd/ from your Web Servers to the NFS server and make sure that each Web Server has its own log directory.
    *sudo umount -f /var/log/httpd*
  
  - Open two ssh/Putty consoles for both Web Servers and run following command:
  *sudo tail -f /var/log/httpd/access_log
  
  Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.
  IMG1
  IMG2
  
  ## Configure Local DNS Names Resolution
  #Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
  img
  
  - Update your LB config file with those names instead of IP addresses.
  BalancerMember http://Web1:80 loadfactor=5 timeout=1
  BalancerMember http://Web2:80 loadfactor=5 timeout=1
  img
  
  curl your Web Servers from LB locally curl http://Web1 or curl http://Web2
  img
  img
  
