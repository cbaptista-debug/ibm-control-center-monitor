


# IBM CONTROL CENTER MONITOR 6.2

O IBM Control Center monitor é um gerenciador de aplicações que realizam entregas como Connect Direct ou B2B Integrator por exemplo .

## Utilizando o banco de dados DB2 para implementação neste caso . 

Provisionamos o banco de dados com uma imagem docker [pip](https://hub.docker.com/r/ibmcom/db2) em um ambiente OpenSuse v15.

```bash
zypper update
```
Install docker on the server:
```bash
zypper install docker
```
Starting docker service
```bash
systemctl start docker
```
Command to climb docker:
```bash
docker run -itd --name mydb2 --privileged = true -p 50000: 50000 -e LICENSE = accept ibmcom / db2

```
To access the instance: 
```bash
docker exec -ti mydb2 bash -c "su - $ {TEST}"
```
Configuration:

Creating bank admin user

```bash
groupadd dasadm1
useradd dasusr1 -m -g dasadm1 -d / home / dasusr1
passwd dasusr1
```
Creating instance owner user

```bash
groupadd db2grp1
useradd db2inst1 -m -g db2grp1 -d / home / db2inst1
passwd db2inst1
```


Creating user to execute user functions (UDFs) and Store procedures
```bash
groupadd db2fgrp1
useradd db2fenc1 -m -g db2fgrp1 -d / home / db2fenc1
passwd db2fenc1

cd /opt/ibm/db2/V11.5/instance/
./db2icrt -u db2fenc1 db2inst1
```
Add user db2inst1 to the visudo
```bash
visudo
Allow root to run any commands anywhere
root ALL = (ALL) ALL
db2inst1 ALL = (ALL) ALL

```
Add to the following lines in the file

```bash
vi / etc / services
db2inst1 50000 / tcp
db2inst1_i 50001 / tcp
```
Command to configure bank license

```bash
cd /opt/ibm/db2/V11.5/adm/
./db2licm -a <Path_License>
```
View the instance settings:

```bash
su db2inst1
cd /opt/ibm/db2/V11.5/bin/
./db2 get dbm cfg
```
Define the TCP protocol for the service and enable automatic database initialization:
```bash
$ chmod 777 /opt/ibm/db2/V11.5/adm/db2set
$ exit
/opt/ibm/db2/V11.5/adm/db2set db2comm = tcpip
/opt/ibm/db2/V11.5/adm/db2set DB2AUTOSTART = yes

```








## When creating a database, the control center user will be the same user as the bank:

Sets the environment to run db2 on the line:


```bash
~ / sqllib / db2profile
```

Download the create_scc_db.sql file and replace with your DB Name 

```bash
/database/config/db2inst1/sqllib/bin/ -stvf /database/config/db2inst1/create_scc_db.sql
```
Start the db2 bench

```bash
sudo chmod 777 /opt/ibm/db2/V11.5/adm/db2start
/opt/ibm/db2/V11.5/adm/db2start
```
[Doc]: (https://www.ibm.com/support/knowledgecenter/pt-br/SS4Q96_6.1.0/com.ibm.help.scc.install.doc/SCC_Create_DB2_DB_From_Scripts.html)


## To configure the IBM Control Center Monitor

First, we need to create the KeyStore and TrustStore so that the application has security encryption.

Execute the commands below in the Keytool path

Default: / opt / IBM / SterlingControlCenter / jre / bin

Create Keystore

Please make sure to update tests as appropriate.


```bash
./keytool -genkey -alias mydomain -keyalg RSA -keystore KeyStore.jks -keysize 2048

./keytool -certreq -alias mydomain -keystore KeyStore.jks -file mydomain.csr

```
Create TrustStore

```bash
./keytool -export -alias mydomain -file client.cer -keystore KeyStore.jks

./keytool -import -v -trustcacerts -alias mydomain -file client.cer -keystore truststore.ts
```
It is also necessary to copy the bank's connection files to make a remote connection.


```bash
chmod + x db2jcc4.jar
chmod + x db2jcc_license_cisuz.jar
```
Standard paths

/opt/IBM/SterlingControlCenter/jre/bin/db2jcc4.jar
/opt/IBM/SterlingControlCenter/jre/bin/db2jcc_license_cisuz.jar

Access the directory>

```bash
cd /opt/IBM/SterlingControlCenter/bin
```
Run the script
```bash
./configCC.sh
```
IBM Sterling Control Center - Not configured ... 1. IBM Sterling Control Center Director
2. IBM Sterling Control Center Monitor
3. All Products
Choose Product Option based on your entitlement [0]: 2

Select option 2.

Inform the KeyStore and Trustore paths, as well as the database “.jar” files.

```bash
/opt/IBM/SterlingControlCenter/jre/bin/KeyStore.jks
/opt/IBM/SterlingControlCenter/jre/bin/truststore.ts
```
Do you want to configure the keystore and truststore files? (Y / N) y

Are you sure about your selection? (Y / N) y

Keystore and truststore configuration ...
Provide the path to your java keystore file [../conf/security/CCenter.keystore]:

```bash
/opt/IBM/SterlingControlCenter/jre/bin/KeyStore.jks
```








These are the aliases found in the keystore /opt/IBM/SterlingControlCenter/jre/bin/KeyStore.jks
[mydomain]


Enter Alias ​​for Key: [mydomain]:

Provide the path to your trust store file [../conf/security/Ccenter.truststore]:

```bash
/opt/IBM/SterlingControlCenter/jre/bin/truststore.ts
```
Provide the following database parameters ...
Provide a database type (DB2 or DB2zOS or Oracle or MSSQL) [DB2]:

DB2

Provide the full path to db2jcc.jar or db2jcc4.jar including the file name):

```bash
/opt/IBM/SterlingControlCenter/jre/bin/db2jcc4.jar
```
This jar file requires one of the following license jar files.
[db2jcc_license_cu.jar, db2jcc_license_cisuz.jar]
Provide the full path to the license file including the file name):

```bash
/opt/IBM/SterlingControlCenter/jre/bi/db2jcc_license_cisuz.jar
```
JDBC Driver Class Name: com.ibm.db2.jcc.DB2Driver Major Version: 4 Minor Version: 27

You provided the following parameters:
Database type = DB2
You provided JDBC driver file (s)
/opt/IBM/SterlingControlCenter/jre/bin/db2jcc4.jar
/opt/IBM/SterlingControlCenter/jre/bin/db2jcc_license_cisuz.jar

Config step: Database connection parameters configuration ...
For detailed IBM Sterling Control Center system requirements, go to the following URL:
http://www-01.ibm.com/support/docview.wss?uid=swg27036103

Provide the following database connection parameters...
Do you want to configure a secure connection to your database? (Y/N) [N] : n

Provide the database host name [127.0.0.1] : Provide the IP of your remote database

Don’t worry with your container ID, by default Docker uses the same port to comunicate
Provide the database port number [50000] : 50000

Provide the database user name [] : dasusr1

Database Password (no blanks):

Re-enter Database Password :

Provide the database name [] :YOURDBNAME

You provided the following database connection parameters:
Database type = DB2
Secure connection to database = N
Database host name = 10.191.6.0
Database port = 50000
Database user name = dasusr1
Database password = ****
Database name = CCENTER

Do you want to partition your database tables? (Y/N)

Y



Config step :  Default User 'admin' Configuration ...
Default user 'admin' password must be set.
Enter Default user 'admin' password (no blanks):
Re-enter Default user 'admin' password :
You provided the following values:
Default user 'admin' password = *****
Enter Default user 'admin' E-Mail address (no blanks): YOUREMAIL
Re-enter Enter Default user 'admin' E-Mail address : YOUREMAIL
Default user 'admin' password and e-mail has been configured.
Default user 'admin' creation complete ...
Config step :  Creating the pre-defined JMS user ...
Pre-defined JMS user creation complete ...
Config step :  Event processor (engine) name configuration ...
Provide a 10 character Event Processor (engine) name [Monitor] : Monitor
You provided the following Event Processor (engine) name :
Event Processor (engine) name  is 'Monitor'
Are the values that were entered correct? (Y/N) [Y]y
Event Processor (engine) name  has been successfully configured ...
Config step :  Engine time zone configuration ...
Default Time Zone : (UTC-03:00) Brasilia)

(UTC-03:00) Brasilia)

(UTC-03:00) Brasilia (BRT/BRST)

(UTC-03:00) Greenland

(UTC-03:00) Santiago

(UTC-03:00) Brasilia (BRT)

(UTC-03:00) Buenos Aires, Georgetown
Choose a time zone number [1] : 1
You chose the following time zone for the IBM Control Center event processor (engine):

(UTC-03:00) Brasilia)
Are the values that were entered correct? (Y/N) [Y]y
Engine Time Zone has been successfully configured ...

Config step : HTTP connector configuration (connection between event processor (engine) and the console)...
HTTP connector configuration ...
Provide a port number. (Enter 0 to disable the HTTP) [58080] : 

58080

Provide a listening address for the above port. [0.0.0.0](0.0.0.0- to listen on all addresses): 

0.0.0.0

You provided the following values:

Port number = 58080

Listening address for port = 0.0.0.0

Are the values that were entered correct? (Y/N) [Y]y
Http Connector configuration complete.
Config step : Secure HTTP connector configuration (connection between Engine and the console)...
Note: A valid keystore is needed for the secure connection.
Do you want to configure the secure HTTP connector? (Y/N)y

Are you sure about your selection? (Y/N)y

Secure HTTP connector configuration ...

Provide a port number.(Enter 0 to disable the HTTPS) [58081] : 

58081

Provide a listening address for the port. [0.0.0.0](0.0.0.0- to listen on all addresses): 

0.0.0.0

You provided the following values:

Port number = 58081

Listening address for port = 0.0.0.0

Are the values that were entered correct? (Y/N) [Y]y

Secure Http Connector configuration has been done successfully!
Config step : Web Application server(Jetty) configuration...
(This step is required for web client and launch page access.)
Note: A valid keystore is needed for the secure connection.

Jetty Web Application server configuration ...
Provide a port number. (Enter 0 to disable) [58082] : 58082

Provide a secure port number. (Enter 0 to disable) [0] : 0
Provide the host name of the event processor (engine). [10.191.1.118] : 

10.191.1.118
Provide a listening address for the above port. [0.0.0.0](0.0.0.0- to listen on all addresses): 0.0.0.0

Do you want the Web App Server to automatically stop when the IBM Control Center event processor (engine) stops? (Y/N)[Y] : y

You provided the following values:

Web Application server port = 58082

Web Application server secure port = 0

Event processor (engine) host name = 10.191.1.118

Listening address for port = 0.0.0.0

Automatically stop the web application server when the event processor (engine) is stopped = Y

Are the values that were entered correct? (Y/N) [Y]y
Jetty Web Application server configuration complete.


Event repository configuration ... -------------------------------------------------------------------- 

Do you want to enable authentication for the Event Repository? (Y/N) [N] : y

Are you sure about your selection? (Y/N)

y 

InstallationInfo.properties updated for EVENT_REPOSITORY_AUTH with true

-------------------------------------------------------------------- 

Config step : Email (SMTP) server configuration ... -------------------------------------------------------------------- 

Important: To create additional Control Center users, you must provide a valid, working email server connection details.Email host name? [localhost] : localhost 

Email port number? [25] : 25 

Email user name? Enter dot (.) for none. [] : . 

Enter user password (no blanks). Enter dot (.) for none.  

Re-Enter user password (no blanks). Enter dot (.) for none.

Email from address? [noone@anywhere] : 

noone@anywhere 

Designated Administrator email address? 

[YOUREMAIL] : YOUREMAIL
You provided the following email configuration options: Email host name = localhost 

Email port number = 25

 Email user name = 

 Email password = **** 

Email from address = noone@anywhere 

IBM Control Center's designated administrator email address = 

YOUREMAIL

Checking whether the specified email server (localhost) is listening (on port 25) or not... Specified Email(SMTP) server(localhost) is alive(at port25)...Are the values that were entered correct? (Y/N) [Y]y 

Updating application.properties with SMTP info... The email configuration is completed successfully!

-------------------------------------------------------------------- 

Config step : JMS configuration ... -------------------------------------------------------------------- 

Do you want to enable JMS events? (Y/N) [n] : n

Are you sure about your selection? (Y/N)y 

JMS successfully configured ...-------------------------------------------------------------------- 

Config step : External Authentication Server configuration ... --------------------------------------------------------------------

Do you want to configure External Authentication Server connection settings(Y/N)?n

Are you sure about your selection? (Y/N)y 

External Authentication Server connection settings completed successfully!

The IBM Control Center event processor (engine) configuration is complete.Updating permissions for encryption key files... Updating permissions for encryption key files...Done


#After making all the configurations and partitioning the database as IBM recommends, we can start the application, for this follow the steps below:

```bash
cd /opt/IBM/SterlingControlCenter/bin
```
Run the script below:

```bash
./runEngine.sh
```
After
```bash
./startWebAppServer.sh
```
To access just type in the browser:

http: // ipdoservidor: 58082

## Author’s

Caique Vicente Baptista 

Diego Crisostomo
