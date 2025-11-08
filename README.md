

# $${\color{red} \textbf{Project}: \textbf{3-tier} \ \textbf{Student} \ \textbf{App}}$$

<img width="1360" height="665" alt="image" src="https://github.com/user-attachments/assets/771929a4-99d0-4f68-a19a-40285bf8972e" />

## $\color{green} \textbf{Prerequisite:}$
- VPC 
- Subnets
- Route Table 
- Nat Gateway
- Internet Gateway
- RDS 

### $\color{blue} \textbf{Create \textbf VPC }$
- Name: VPC-3-tier
- CIDR: 192.168.0.0/16

### $\color{blue} \textbf{Create \ Subnets}$
1.Subnet-1
- Name: Public-Subnet-Nginx
- CIDR: 192.168.1.0/24

2.Subnet-2
- Name: Private-Subnet-Tomcat
- CIDR: 192.168.2.0/24

3.Subnet-3
- Name:Private-Subnet-Database
- CIDR: 192.168.3.0/24

  
### $\color{blue} \textbf{Create \ Internet \ Gateway }$
- Name: IGW-3-tier
- attach igw to vpc

### $\color{blue} \textbf{Create \ Nat \ Gateway }$
- Name: NAT-3-tier
- create in public subnet

### $\color{blue} \textbf{Create \ Route \ Table }$
1. RT-Public-Subnet
   - add public subnet
   - add igw
2. RT-Private-Subnet
   - add private subnets
   - add nat

![vpc-flow](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/aaae3b25-2030-453e-992b-c4ae9a2291b9)

### $\color{blue} \textbf{Create \ EC2 \ Instances }$

1. Nginx-Server-Public  ->create in public subnet ->allow port = 80,22
2. Tomcat-Server-Private ->create in private subnet ->allow port = 8080,22
3. Database-Server-Private ->create in private subnet ->allow port = 3306,22

![instances](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/e2080675-51d3-4101-824a-81129830187e)


### $\color{blue} \textbf{Create \ Database \ In \ RDS  }$
- Go To RDS
- Created Database
- Standard create 
- Free tier 
- DB name – database-1 
- Username – admin 
- Password – Passwd123$
- VPC –  VPC-3-tier
- Connect to Instance -> choose database instance
- Public access – no 
- A.Z. – no preference 
- Create database 
- Edit security group -> Add 3306 port

![database](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/31d6e98c-986a-41ad-bbe8-0418e9beaa17)

### $$\color{green} \textbf {Connect To Nginx-Server-Public }$$

![nginx-server](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/0cbb6175-9570-4ce7-a4ef-4cd7b5b24ceb)

- connect to instance
- change hostname
  
- create file with name 3-tier-key.pem
  ```` 
  vim 3-tier-key.pem
  ````
- copy private key and paste it here
 
### $$\color{orange} \textbf {Now SSH  into Database  Server}$$

![database-instance](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/8159a278-d612-441e-93da-d581428cdd3a)

````
sudo -i
````
````
yum install mariadb105-server -y
````
````
systemctl start mariadb
````
````
systemctl enable mariadb
````
### $\color{orange} \textbf{Log \ in \ into \ database}$

![login into database](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/ba0c082a-060f-48f9-8520-83c906337251)

````
mysql -h rds-endpoint   -u admin -pPasswd123$
````
Note: replace rds-endpoint with actual endpoint value

````
show databases;
````
````
create database  studentapp;
````
````
use studentapp;
````

### $\color{blue} \textbf{Run \ this \ query \ to \ create \ table:}$
````
 CREATE TABLE if not exists students(student_id INT NOT NULL AUTO_INCREMENT,  
	student_name VARCHAR(100) NOT NULL,  
	student_addr VARCHAR(100) NOT NULL,   
	student_age VARCHAR(3) NOT NULL,      
	student_qual VARCHAR(20) NOT NULL,     
	student_percent VARCHAR(10) NOT NULL,   
	student_year_passed VARCHAR(10) NOT NULL,  
	PRIMARY KEY (student_id)  
);
````
````
show tables;
````
Logout from database:
````
exit
````
#### $${\color{green} \textbf {Back to nginx-server}}$$
  
### $${\color{magenta} \textbf{Now SSH into Tomcat Server}}$$

![tomcat-server](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/a6cd6922-7be0-4f6b-ab56-d69423093ae5)

- ssh -i 3-tier-key.pem  ec2-user@ip-of-tomcat-vm
````
sudo -i
````
````
yum install java -y
````

````
curl -O https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.109/bin/apache-tomcat-9.0.109.tar.gz
````
````
tar -xzvf /apache-tomcat-9.0.109.tar.gz -C /opt/
````
**go to webapps dir and download .war file(application)**
````
cd /opt/apache-tomcat-9.0.109/webapps
````
````
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/student.war
````
````
cd ../lib
````

````
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/mysql-connector.jar
````
### $\color{magenta} \textbf{ MODIFY \ context.xml}$
````
cd apache-tomcat-9.0.107/conf
````
````
vim context.xml
````
add below line [connection string] at line 21
````
 <Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="USERNAME" password="PASSWORD" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://DB-ENDPOINT:3306/DATABASE-NAME"/>

````
![image](https://github.com/user-attachments/assets/1dd0a494-6e94-4513-9164-688bd2860b48)


````
cd ../bin
````
````
chmod +x catalina.sh
````
````
./catalina.sh start
````


````
exit
````
### $${\color{green} \textbf {Back to nginx-server}}$$

````
sudo -i
````
````
  yum install nginx -y
````
````
 vim /etc/nginx/nginx.conf
````
- :set nu
(enter below data in line 47 in between error and location)
````
location / {
proxy_pass http://private-IP-tomcat:8080/student/;
}
````
![image](https://github.com/user-attachments/assets/31feb10d-d005-4240-b629-839b8a596777)

- :wq  ->save file

````
systemctl start nginx
````

## $\color{red} \textbf{Go \ To \ Browser \ Hit \ Public-IP \ Nginx}$
![nginx-output](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/b2929899-fee8-4790-8178-c870faa55124)

![register-students](https://github.com/abhipraydhoble/Project-3-tier-Student-App/assets/122669982/210a4bef-8fc2-4ada-9faa-ad3f8b751958)
