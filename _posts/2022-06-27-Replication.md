---
title: "Master-slave replication in Mysql"
date: 2022-06-27T15:34:30-04:00
categories:
  - Replication
tags:
  - Mysql Replication
---

# Setup 2 nodes master slave replication in VM machine
   MySQL replication is a real-time mechanism and one-way process that
enables data  to be copied automatically from one MySQL database server (the master) to another one or more MySQL
database servers (the slaves) as backup server.

This allows us to create a continuous live backup of the database
and can switch over the slave database and keep the
application up and running when there is down due to any difficulties.

Mysql natively provides an easy way to replicate data from one server to another server.

## Prerequisities
1)  Two machines with CentOs 7 installed in Virtual machine 

2) Install  Mysql  on both machines(i.e. master and slave server)

The Centos address of Master is <b>192.XXX.XX.XXX</b>, which acts as  master database server.

The Centos address of slave is <b>192.XXX.XX.YYY</b>, which acts as slave database server.
 

 ## Configuration and environment setup
  We need to configure Firewall for database access. If firewall is disabled ,we can skip otherwise we need to apply the TCP Port 3306 rule that adds a Firewall Policy and  allows traffic on port 3306(MySQL default port) .

  For root login we  can have command <b>sudo su </b> as
  ```bash
  ram@ram-Inspiron:~$ sudo su
```
   This command will put  into a root environment and it will prompt us for our user password


  For view status:
  ```bash 
  root@localhost #systemctl status firewalld
  ```

  Stop firewall:  
  ```bash  
  root@localhost #systemctl stop firewalld.service
  ```
  



Change SELINUX to permissive using the below command
``` bash
root@localhost # vi /etc/selinux/config
```
```
#This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX= permissive
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```


### Network connection 
My setup  in VirtualBox uses two NICs.  The first uses "host-only" connection that allows my host and other virtual machines to interact. 

VirtualBox can create several of these virtual host-only networks.<br/>
![Host-only adapter](/assets/images/host_only.png)
<br/>
To create a host-only connection in VirtualBox, start by opening the preferences in VirtualBox. Go to the "Network" tab, and addd a Host-only Network.Also,  use <b>ALLow All</b> tab for <b>Promiscuous Mode</b>





The second is a NAT to allow the box to communicate with the outside world through my host computer’s network connection. 
(NAT is the default, so shouldn't require any setup.).We'll be able to download packages,check email,browse the website and so on. 

![](/assets/images/NAT.png)


Next, assign this NAT adapter to the virtual machine. Select the VM and press "Settings". Go to the "Network" tab, and select "Adpater 2". Enable the adapter, set it to a "NAT".

### For connection between host and VM's:

I did ping and telnet to test whether two machines  communicate  or not .

In the master server : <code>ping <b>192.XXX.XX.YYY(slave ip)</b> </code>,     <code>telnet <b>192.XXX.XX.YYY(slave ip)</b> 3306 </code>

In the slave server : <code>ping <b>192.XXX.XX.XXX(master ip)</b> </code> , <code>telnet <b>192.XXX.XX.XXX(master ip)</b> 3306 </code>

If there is data returned then we can be sure there is a connection.

 


### Installation
 
At first , we need to  install and configure MySQL 5.6 on both master ans slave server.

Following   are the  below steps to install MySQL Server 5.6 .
 <b>Note : we must  login with root user .</b>

 Here, we can  to connect our local host to the Virtual Machine we  created using SSH as :
``` bash
 ssh <username on the Virtual machine>@< hostname or IP>
 ```
 ``` bash
 For example: ssh master@192.168.XX.XXX
```


then , we begin to install mysql 5.6 in master server .
``` bash
ssh master@192.168.XX.XXX
```
after login it prompt the user for password .
then , we get login to master server 

Steps :
              
  
By default, CentOS 7 and RHEL 7 have MariaDB. However, If we want to use MySQL then we need to remove the MariaDB then install MySQL packages.

 * Remove MariaDB packages by using yum remove.

``` bash 
[root@localhost ~]# yum remove mariadb mariadb-server -y
```
 * Download the rpm package, which will create a yum repo file for MySQL Server installation.
``` bash
[root@localhost ~]#yum install wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

* Install this downloaded rpm package by using rpm command
``` bash
[root@localhost ~]#rpm -ivh mysql-community-release-el7-5.noarch.rpm
``` 

* After the installation of this package. We will  verify it whether  new yum repo related to MySQLis installed or not.
``` bash
[root@localhost ~]# ls -1 /etc/yum.repos.d/mysql-community*
``` 

* Now we will install MySQL Server 5.6 . All dependencies will be installed
``` bash 
[root@localhost ~]#yum install mysql-server
 ``` 

Then if successfully installed we can assume mysql is installed in our server.

 * To start MySQL Service, 
 ``` bash 
  [root@localhost ~]# systemctl start mysqld
``` 
 *  To stop MySQL Service,
 ``` bash  
 [root@localhost ~]# systemctl stop mysqld
``` 
 *  To restart MySQL Service
 ``` bash 
 [root@localhost ~]#systemctl restart mysqld
``` 
 *   To get status of MySQL Service
``` bash 
 [root@localhost ~]#systemctl status mysqld
``` 
 *  then for secure  installation we use the below command
 ``` bash 
 [root@localhost ~]#mysql_secure_installation
 ``` 

 This will prompt us  for the default root password.
 On fresh installation of MySQL Server. The MySQL root user password is left blank.
  As soon as we entered  for blank password , again we are  required to change for a new password. 

After the change of  password , we would   <b> press Y</b> and then <b>ENTER </b>to all the subsequent questions in order to remove anonymous users, disallow remote root login, remove the test database and access to it, and reload the privilege tables.



10.  We can verify our installation by  connecting  to MySQL as root (-u root), prompt for a password (-p), as 
 ``` bash 
[root@localhost ~]# mysql -u root -p
 ``` 
this will prompt the mysql root password .
We can verify mysql version using the command as :
 ``` bash 
mysql -V 
 ``` 
 or 
  ``` bash 
  mysql --version
   ``` 

Here, we’ve installed and secured MySQL on a CentOS 7  master server.
In the same way, we need to install and configure mysql in slave server too.




To configure replication you should have at least  two servers.

Master Server : 192.xxx.xx.xx

Slave Server: 192.xxx.xx.yy

 # MASTER SERVER CONFIGURATION:

We need to setup these steps to configure Master.

We need to provide unique  server ID,  enable log-bin, bind-address  with the  IP address 

<b>Master-server</b>

step 1. Open MySQL config file using following command
 ``` bash
 [root@localhost ~]# vim /etc/my.cnf
  ```
 Add the following information to the bottom of the <b>[mysqld] </b> section
 to add  and edit then following information. Here, we  need to press <b> keyword I </b> for information insertion 
``` bash
server-id = 1
log_bin =mysql-bin
bind-address = MASTER_IP_ADDRESS
```

After adding , we need to save and exit 
Press Ctrl+c and then type <b>:wq! </b>to save and exit the configuration file.


step 2: After changes in configuration file, now restart MySQL service.
``` bash
# systemctl restart mysqld
```

step 3: Login to MySQL server in master server and  execute the following commands to create a user.

``` bash
[root@localhost ~]# mysql -u root -p
mysql> CREATE USER ‘user_slave’@’SLAVE_IP_ADDRESS‘ IDENTIFIED BY ‘SLAVE_PASSWORD‘;
mysql> GRANT REPLICATION SLAVE ON *.* TO ‘user_slave’@’SLAVE_IP_ADDRESS‘;
mysql> FLUSH PRIVILEGES;
```

step 4 :

we need to note down to record the current log position so that our Slave server can start reading data from that log position. To know the current status of master server, execute the following command 

Check binary log file and position details  in MySQL master server.
``` bash 
mysql > show master status\G;
```

figure
``` bash 
+-------------+--------------------+--------------+------------------+-------------------+
| File        |Position             | Binlog_Do_DB | Binlog_Ignore_DB | |
+-------------+--------------------+--------------+------------------+--------
mysql-bin.0001|             332    |              |                  |                   |
+-------------+--------------------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

Note: we need to note down binary position details  to configure in Slave Server.

<b>CONFIGURE THE SLAVE</b>


After  installed MySQL on slave, we opened the MySQL configuration file to update some configuration.
 
 Step 1: Open MySQL config file using following command
 ``` bash
 [root@localhost ~]# vim /etc/my.cnf
  ```
 Add the following information to the bottom of the [mysqld] section
 to add  and edit then following information need to press I for information insertion 
``` bash
server-id = 2
log_bin =mysql-bin
bind-address = SLAVE_IP_ADDRESS
```
step 2: After changes in configuration file, now restart MySQL service.
``` bash
[root@localhost ~]# systemctl restart mysqld
```

3. Now,  we need to configure slave server parameters which will use to connect with the master server.
``` bash 
[root@localhost ~]# mysql -u root -p
mysql> stop slave;
mysql> CHANGE MASTER TO MASTER_HOST='192.168.xx.xxx'(master ip), 
       MASTER_USER='user_slave',
       MASTER_PASSWORD='SLAVE_PASSWORD',
       MASTER_LOG_FILE='mysql-bin.0001',
       MASTER_LOG_POS=332;
mysql> start slave;
mysql> exit;
```

Here,we've  set up a master-slave replication in MySQL server 5.6.

<b>Test MySQL Replication</b>

Foremost task to verify the master-slave replication following command needto be executed.
``` bash
mysql> SHOW SLAVE STATUS\G;
```
``` bash 
<MariaDB [ledcontrol]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: 192.168.xx.xxx
                  Master_User: repl_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.0001
          Read_Master_Log_Pos: 666
               Relay_Log_File: mysqld-relay-bin.000009
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.0001
             Slave_IO_Running: Connecting
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 332
              Relay_Log_Space: 1245
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
              Last_IO_Errno: 1045
              Last_IO_Error: error connecting to master 'repl_user@192.168.xx.xxx:3306' - retry-time: 60  maximum-retries: 86400  message: Access denied for user 'repl_user'@'192.168.xx.xxx' (using password: YES)
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
```
There may occur error where slave can't connect to master as:
```bash
Last_IO_Error: error connecting to master 'repl_user'@xx.xx.xx.xx:3306' - retry-time: 60 retries: 1 message: Can't connect to MySQL server on 'xx.xx.xx.xx:3306' (113)
Slave_IO_Running: Connecting
Slave_SQL_Running: Yes
```
 to fix this error ,we have following command in slave server
``` bash 
STOP SLAVE;
SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1; /*This is to skip the duplicate record*/
CHANGE MASTER TO GET_MASTER_PUBLIC_KEY = 1;
START SLAVE;
```
Then ,we can test the replication by creating a test database on master server. As a result, it should appear automatically on slave

To create a test databases on master, Login to MySQL and execute the following command.
``` bash
root@mysql-master:~# mysql -uroot -p;
mysql> CREATE DATABASE testdb;
```
Now, Open up MySQL command line interface on your <b>slave server </b>and check if the new database exist.

To get the list of database, execute the following command on slave.
``` bash
root@mysql-slave:~# mysql -uroot -p;
mysql> SHOW DATABASES;
```

We must see a database named “testdb”. If we  can see the database, MySQL replication is working. If not, Replication is not working.

Thus, this all about the setup for the Mysql master slave replication.


# References
1.https://christophermaier.name/2010/09/01/host-only-networking-with-virtualbox/

2.https://www.interserver.net/tips/kb/create-master-slave-replication-mysql-server/

3.https://www.linoide.com/how-to-install-mysql-5-6-on-centos-7/

4.https://www.virtualbox.org/manual/ch06.html
