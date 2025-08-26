# Three - Tier Java web application Deployment 
####  This project focuses on deploying a secure, scalable, and modular web application using a Three-Tier Architecture on Amazon Web Services (AWS). The architecture separates concerns into three layers: Proxy Layer (NGINX), Application Layer (JAVA TOMCAT), and Database Layer (MySQL). This division ensures better maintainability, flexibility, and security. The deployment process involved creating a custom Virtual Private Cloud (VPC), launching EC2 instances, setting up secure communication channels.In VPC 1 Public subnet for _Proxy server_ and 2 Private Subnets for _Application and Database server_ for making them secure.Use IGW and NAT Gateways to communicate the Server.
- **Proxy Server**: Nginx Server (acts as a reverse proxy)
- **Application Server**: Java Web Application running on Tomcat
- **Database Server**: MySQL Database with RDS
##  Project Details

- **Cloud Provider**: AWS
- **Network**: Custom VPC with 1 public and 2 private subnets
- **Frontend Server**: Nginx Server
- **Backend Server**: Tomcat running a Java 
- **Database Server**: RDS
- **OS**: Amazon Linux 
## Three-Tier Architecture
![alt text](<image/Your paragraph text.png>)
## 🛠️Steps to Deploy 
## Step 1) Create a VPC network ( Virtual Private cloud )
- Go to _AWS_ console and in _search bar_ , Search ***VPC*** and click on that VPC
- After going into VPC ,Click on `Your VPCs` in the left side panel, to create VPC
- In ` Your VPCs ` , click on ` Create VPC  `
- In create VPC ,VPC settings make some configuration
```bash
# VPC Settings
--> Resources to create : VPC only
--> Name tag : three-tier-vpc    # give name to VPC
--> IPv4 CIDR block : IPv4 CIDR manual input    # kept as it is no change
--> IPv4 CIDR : 10.0.0.0/16       # Give a CIDR range to VPC
--> And OTHER configuration kept as it is NO CHANGE
--> and Click on ( Create VPC )
```
- In left panel `VPC DASHBOARD` ,click on ***Filter by VPC*** and select our Cretaed VPC
![alt text](<image/Screenshot 2025-07-09 193703.png>)
## Step 2) Create Subnets, One Public and TWO Private
- Below the _Your VPCs_ their is ` Subnets `, click on that to create subnets,
- In Create VPC ,
```bash
# VPC
--> In VPC ID : mention our ( Three-tier-vpc ) which we created

# Subnet Settings, 
# in subnet settings we have to create three subnets

# for 1 subnet
--> subnet name : Public-subnet
--> Availability zone : select one AZ
--> IPv4 VPC CIDR  block : default it come from VPC ( 10.0.0.0/16 )
--> IPv4 subnet CIDR block : 10.0.0.0/20   # we have calculate the CIDR range of subnet
--> click on add new 

# for 2 subnet
--> subnet name : Private-subnet-1 
--> Availability zone : select one AZ
--> IPv4 VPC CIDR  block : default it come from VPC ( 10.0.0.0/16 )
--> IPv4 subnet CIDR block : 10.0.16.0/20   # we have calculate the CIDR range of subnet
--> click on add new 

# for 3 subnet
--> subnet name : Private-subnet-2 
--> Availability zone : select one AZ
--> IPv4 VPC CIDR  block : default it come from VPC ( 10.0.0.0/16 )
--> IPv4 subnet CIDR block : 10.0.32.0/20   # we have calculate the CIDR range of subnet
--> click on- Create Subnet

```
- After creating subnets ***select the Public Subnet and click on Actions***,and ***select Edit subnet settings*** and In ***Auto-assign IP settings***, just click on ***Enable auto-assign public IPv4 address*** and click on save settings
- This auto-assigning settings for only Public Subnet to making it Public to accept the requests.
![alt text](<image/Screenshot 2025-07-09 194051.png>)

## Step 3) Create Internet Gateways( IGW )
- In Left panel under Routes tables, click on `Internet Gateways `, After clicking on that click on `create Internet Gateways`
- Inside create internet gateways 
```bash
# Internet Gateways settings 
--> Name tag : IGW-3tier
# Tags-optional
--> click on , Cretae internet gateways
```
![alt text](<image/Screenshot 2025-07-09 194151.png>)
- After creating IGW it is in Detached State now attach to VPC, _attach to VPC_
- In right side of IGW console ,click on `Actions` and then click on `Attach to VPC`
- Inside the Attach to VPC ,
```bash
# Attach to VPC
--> VPC : select the VPC which we created( three-tier-vpc )
--> click on , Attach internet Gateway
```
## Step 4) Create NAT Gateway
- In left panel vpc, click on `NAT Gateways`
- In NAT gateway, click on `Create NAT gateway`
- Inside create NAT gateway,
```bash
# NAT gateway settings

--> Name : NAT-3tier
--> Subnet : SELECT public-subnet
--> Connectivity type : public
--> Elastic IP allocation ID : CLICK ON ( Allocate Elastic IP )
--> click on ( Create NAT gateway )
```
- After creating NAT gateway Add  it into Private Route table 
## Step 5) Edit Routes and Create One Private Route Table
- In left panel click on `Routes Tables`
- inside Route Table , One defult Public route table are Created with our VPC 
- click on _Route table ID_of that Public-RT and below it select `Routes` , in that Routes click on ` Edit Routes` 
- Inside Edit routes
```bash
# Edit Routes
--> Click on , Add route 
--> Destination : 0.0.0.0/0 
--> Target : Internet Gateway and select the IGW created
--> click on save changes
```
![alt text](<image/Screenshot 2025-07-09 194927.png>)

#### Create Private Route Table 
- Go to Route tables and click on `Create route table`
- Inside create route tables
```bash
# Route table settings

--> Name : private-RT
--> VPC : SELECT CRAETED VPC ( three-tier-vpc )
--> Click on Create Route Table
```
- Then click on _Route table ID_ of private-RT route table and
- below it select `Routes` , in that Routes click on ` Edit Routes` 
- Inside Edit routes
```bash
# Edit Routes
--> Click on , Add route 
--> Destination : 0.0.0.0/0 
--> Target : NAT Gateway and select the NAT gateway which we created
--> click on save changes
```
- after that we have to associate private subnet,
- for that click on `Subnet Associations` right side of Routes, click on them
- In Explicit subnet association click on `Edit subnet associations`, 
- Inside the Edit subnet assoctions, in `Available  subnets` SELECT our two Private subnet ,( Private-subnet-1 and Private-subnet-2 )
- then click on `Save association`
![](<image/Screenshot 2025-07-09 195218.png>)
## Step 6) Create security Group
- In search bar search _EC2_
- In EC2 console, in left panel in `Network & Security` click on `Security group`
- In security group click on `Create security group`
- Inside create security group,
```bash
# Basic details
--> Security group name : 3-Tier-SG
--> Description : allow ssh http tomcat mysql
--> VPC : three-tier-vpc

# Inbound rules
click on Add rule

--> Type :SSH           Port range :22     Source :Anywhere-IPv4
--> Type :HTTP          Port range :80     Source :Anywhere-IPv4
--> Type :MYSQL/Aurora  Port range :3306   Source :Anywhere-IPv4
--> Type :Custom TCP    Port range :8080   Source :Anywhere-IPv4

# clcik on create security group
```
![alt text](<image/Screenshot 2025-07-09 195525.png>)
## Step 7) Create RDS 
- In search bar , search **RDS** and and click on Aurora and RDS
- In left side of RDS console, click on `Databases`
- Click on ` create Database`
- Inside create database
```bash
# Create Database
--> choose a database creation method : standard create

# Engine options
--> Engine type : SELECT MariaDB
-->other kept default

# Templates
--> Templates : Dev/Test - for free tier

# settings
--> DB cluster identifier : three-tier-rds
# credentials settings
--> Master username : NO CHANGE 
--> Credentials management : SELECT Self managed
--> click on Auto generate password  

# Cluster storage configuration 
--> configuration options : NO CHANGE

# Instance configuration
--> DB-class :All configuration kept as it is 

# Availability $ Durability
--> multi-AZ deployment : SELECT Dont create an Aurora Replica 

# Connectivity
--> compute resource :Dont connect to an Ec2 compute resource
--> Network type : IPv4
--> In VPC : SELECT CREATED VPC( three-tier-vpc )
--> DB subnet group  : select created subnets
--> Public access : NO
--> VPC security group : choose existing
--> Existing VPC security group : 3-Tier-SG
--> AZ : select AZ
--> other no change kept as it is

--> Click on  create Database
```
- after DB created copy the password which come in the notification of RDS console, Open it and copy the password paste in your notepad ***It only reflect one Time*** 
- It take 5-10 min to come in _AVAILABLE_ state
![alt text](<image/Screenshot 2025-07-09 203633.png>)
## Step 8) Create Instance 
- Search for ***EC2*** in search bar 
- we have to created three server in EC2 
- inside EC2 , click on `Instance` and click on `Launch instances` 
- Inside Launch instance, 
### i) First create ***proxy-server***
```bash
# Launch an instance
--> Name : proxy-server
--> AMI & OS : Amazon Linux 2 ( free-tier )
--> Instance type : t2.micro ( free-tier )
--> key pair : SELECT EXISTING KEY ( key.pem )

# Network settings
click on EDIT
--> VPC : three-tier-vpc
--> Subnet : public-subnet
--> AZ : select one AZ
--> auto-assign public IP : Enable
--> fierwall : select existing security group
--> Security group name : 3-Tier-SG    # created SG
--> other kept as it is NO CHNAGE

```
- click on `Launch instance`
### ii) Second create ***app-server***
```bash
# Launch an instance
--> Name : app-server
--> AMI & OS : Amazon Linux 2 ( free-tier )
--> Instance type : t2.micro ( free-tier )
--> key pair : SELECT EXISTING KEY ( key.pem )

# Network settings
click on EDIT
--> VPC : three-tier-vpc
--> Subnet : private-subnet-1
--> AZ : select one AZ
--> auto-assign public IP : Disable
--> fierwall : select existing security group
--> Security group name : 3-Tier-SG    # created SG
--> other kept as it is NO CHNAGE

```
- click on `Launch instance`
### iii) Third create ***db-server***
```bash
# Launch an instance
--> Name : db-server
--> AMI & OS : Amazon Linux 2 ( free-tier )
--> Instance type : t2.micro ( free-tier )
--> key pair : SELECT EXISTING KEY ( key.pem )

# Network settings
click on EDIT
--> VPC : three-tier-vpc
--> Subnet : private-subnet-2
--> AZ : select one AZ
--> auto-assign public IP : Disable
--> fierwall : select existing security group
--> Security group name : 3-Tier-SG    # created SG
--> other kept as it is NO CHNAGE

```
- click on `Launch instance`
![alt text](<image/Screenshot 2025-07-09 200132.png>)

## Step 9) Set up the Proxy-server
- Connect the Proxy Server 
- Copy the command line from `SSH client`
- Paste on Git bash and enter
![alt text](<image/Screenshot 2025-07-09 200236.png>)
- change hostname to proxy-server ( its optional )
```bash
sudo hostnamectl hostname proxy-server
```
- now `exit` ,to copy KEY on proxy server to connect the Private SERVERS 
![alt text](<image/Screenshot 2025-07-09 200512.png>)
- Then go to proxy-server

### Install Nginx on Proxy server
```bash
# switch to root
sudo -i

# update the server
yum update

# install nginx
yum install nginx -y
systemctl start nginx
systemctl enable nginx

cd /etc/nginx/
vim nginx.conf

# inside nginx.conf
#enter ( i ) to insert or write
# scrol up to server , In server add location of app server
location / {
proxy_pass http://(Private-IP-of-Application-server):8080;
}
# press (esc) and then ( :wq ) to save and exit
```
![alt text](<image/Screenshot 2025-07-09 201052.png>)
## Step 10) Set up App-server
```bash
# go to ec2-user/ 
cd

# Now go to App-server to configure Java application on tomcat
ssh -i < key > ec2-user@<Private-IP-of-app-server>
enter
```
![alt text](<image/Screenshot 2025-07-09 201307.png>)
```bash 
# to change hostname to app-server
sudo hostnamectl hostname app-server
exit
# repeat ssh step to connect app-server
```
### Install Java on App-server
```bash
# switch to root 
sudo -i

#update server
yum update 

# install java
yum install java -y

#change directory opt/
cd /opt
#install tomcat 9
curl -O https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.107/bin/apache-tomcat-9.0.107.tar.gz

ls 
tar -xvzf <downloaded file name>   # extract the file
# rename the extract file
mv <extract file>/ <new name>
ls

# go inside extract file
cd < apache- a new name given to file>
ls
# go inside bin/
ls
./catlina.sh start    # to start the tomcat service
# copy instance public ip and hit on browser here we can access your tomcat page
# public Ip:8080

cd ..
cd ..
# now store your java code to webapps/ folder
cd webapps/
# download code
curl -O https://s3-us-west-2.amazonaws.com/studentapicit/student.war
ls
```
## Step 11) Check that Java app deployed or not successfully
- Now go back to _proxy-server_ to give the path of application 
- In proxy-server `cd /etc/nginx/` and edit the nginx.conf file `sudo vim nginx.conf` 
- inside nginx.conf
- enter ( i ) to insert or write
scrol up to server , In server add location of app server
`location / {
proxy_pass http://(Private-IP-of-Application-server):8080/student/;
}`
- press (esc) and then ( :wq ) to save and exit
![alt text](<image/Screenshot 2025-07-09 202420.png>)
- restart nginx `sudo systemctl restart nginx`
- now go to browser and hit public ip and fill the form and register
![alt text](<image/Screenshot 2025-07-09 203414.png>)
- But this values are not stored,because of database that,we not set up
![alt text](<image/Screenshot 2025-07-09 202656.png>)

## Step 12) Set up Database - Server
```bash
# go to ec2-user/ 
cd

# Now go to db-server to connect RDS
ssh -i < key > ec2-user@<Private-IP-of-db-server>
enter
```
![alt text](<image/Screenshot 2025-07-09 203122.png>)
```bash 
# to change hostname to db-server
sudo hostnamectl hostname db-server
exit
# repeat ssh step to connect db-server
```
#### Install MYSQL to Database server 
```bash  
# switch to root
sudo -i

# update server
yum update

# install mariadb105-server of MYSQL
yum install mariadb105-server -y
systemctl start mariadb
systemctl enable mariadb

# now connect to RDS

mysql -h <endpoint of RDS> -u admin -p
password :         # In password paste the copied password from RDS and click enter
#inside database 

create database studentapp;
use studentapp;
# create table students
CREATE TABLE if not exists students(student_id INT NOT NULL AUTO_INCREMENT,
 student_name VARCHAR(100)NOT NULL,
 student_addr VARCHAR(100)NOT NULL,
 student_age VARCHAR(3)NOT NULL,
 student_qual VARCHAR(20)NOT NULL,
 student_percent VARCHAR(10)NOT NULL,
 student_year_passed VARCHAR(10)NOT NULL,
 PRIMARY KEY (student_id)
 );

```
![alt text](<image/Screenshot 2025-07-09 212642.png>)
### To connect Java application and Database you required connector
##  Step 13) Download Connector 
- Go back to _app-server_
```bash 
cd /opt/apache/lib
ls
#download connector
curl -O https://s3-us-west-2.amazonaws.com/studentapicit/mysql-connector.jar
ls   # you can see the mysql-connector.jar
```
![alt text](<image/Screenshot 2025-07-09 213518.png>)
- Now configure context.xml file which is in `/opt/apache/conf/context.xml`
```bash
cd 
cd /opt/apache/conf/
sudo vim context.xml
# just change in last 2nd number line
enter ( i ) #to insert or write
<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSourc
 e"
 maxTotal="500" maxIdle="30" maxWaitMillis="1000"
 username="admin" password="12345678" driverClassName="com.mysql.jdb
 c.Driver"
 url="jdbc:mysql://endpoint:3306/studentapp?useUnicode=yes&amp;characte
 rEncoding=utf8"/>
# in this above code just here configure your ( database name-studentapp ),( username ),( password ),( endpoint )
```
- Now restart the tomcat 
```bash
cd /opt/apache/bin
./catlina.sh stop    # first stop the service then start
./catlina.sh start 
```
- Now check that values are Stored or Not
<video controls src="image/Screen Recording 2025-07-09 214434-1.mp4" title="Title"></video>
- Also check in Database
- Go to Database server and check
![alt text](<image/Screenshot 2025-07-09 214626.png>)

---
## So successfully we deploy our Three-Tier java application.
#  Project Summary
The project titled “Three-Tier Java Web Application Deployment in AWS VPC” demonstrates the practical implementation of a secure, modular, and scalable web application architecture using cloud-based virtual servers. The core idea of this project is to deploy a typical three-tier architecture, which separates the application into three functional layers: the Proxy server, Application server, and Database server, each hosted on an independent Amazon EC2 instance within a custom Virtual Private Cloud (VPC). The Proxy server consists of an NGINX Server that serves nginx page and acts as a reverse proxy to forward dynamic requests to the Application server. The Application server hosts a Java web application deployed on an Apache Tomcat server, which handles business logic, processes user input, and communicates with the backend database using mysql-connector.jar connector to connection. The Database server comprises a MySQL database server configured to securely store and manage user data submitted through the application. All three servers are logically segregated using public and private subnets to ensure tight security—only the Proxy Server is exposed to the internet, while the Application and Database Servers reside in private subnets with strict access controls defined through AWS Security Groups. This manual deployment, intentionally done without an Auto Scaling Group (ASG) or Load Balancer, provides the intern with a clear understanding of fundamental cloud networking, server configuration, and secure multi-tier application hosting. The project demonstrates how web applications can be structured to follow industry best practices for modularity and security while using open-source technologies and basic cloud infrastructure. Overall, the implementation bridges theoretical knowledge and real-world deployment, Linux server management, web server setup, and relational database integration in a production-like environment.
