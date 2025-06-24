# Azure-Infra Project: Deploying a Java springboot application with Azure SQL Database on Azure app service
## Steps:
--------
<h2 align="center">Part 1: Setting up Infrastructure</h1>

1. create an resource group named **infra-rg** in canadacentral
2. create a virtual network named infra-vnet in the above resource group
3. create a subnet named app-subnet in the above **infra-vnet**
4. create a ubuntu virtual machine named **InfraVM** within the app-subnet with 8080 and 22 ports open 
5. Login to VM using SSH
6. Install Git
```bash
sudo apt update
sudo apt install git -y
```
7. Install JAVA
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```
8. Install zip
```bash
sudo apt update
sudo apt install zip
```
9. Install Maven and set the path
```bash
sudo apt update
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
tar -xvzf  apache-maven-3.9.9-bin.tar.gz
ls
mv apache-maven-3.9.9 maven
pwd
cd
ls -a
vi .bashrc (an editor will open paste the below)
export MAVEN_HOME=/home/InfraVM/maven
export PATH=$MAVEN_HOME/bin:$PATH
source .bashrc
mvn -v
```
10. Install Azure CLI and login to your account

a. Azure CLI  Installation
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
b. Logging into Azure
```bash
az login --use-device-code
```
c. Check the account
```
az account show
```

11. create an Azure SQL Database manually using portal in **infra-rg** very carefully only with below namings
```bash
sqlserver name: onesqlserver123
Authentication: sql server authentication
username: luckyadmin
passoword: YourSecurePassword@123
Databasename: onepieceDB
```
IMP NOTE: while creating the database make sure you checked **yes** for the Add current client IP address

12. Install SSMS locally and try to login to your database
  - Powershell command to install ssms locally or in windows VM
```bash
powershell -Command "Invoke-WebRequest -Uri 'https://aka.ms/ssmsfullsetup' -OutFile 'SSMS-Setup.exe'; Start-Process 'SSMS-Setup.exe' -ArgumentList '/install', '/quiet', '/norestart' -NoNewWindow -Wait"
```
13. clone the repo: https://github.com/luckysuie/Java-springboot
```bash
git clone https://github.com/luckysuie/Java-springboot
```

14. Local Testings: 
a. Navigate to VM and go to the project folder 
```bash
cd Java-springboot
```
b. Run Maven package
```bash
mvn package
```
c. Navigate to target folder there you will see you jar file and rename it
```
mv login-1.0.jar app.jar
```
d. Run the Jar file in the target directory itself
```bash
java -jar app.jar
```
e. Browse the VM ip address with 8080 port you will see your application running

<h2 align="center">Part 2: Deploying your application to azure app service</h1>

15. Create an Azure app service using below
a. Create App service plan
```bash
az appservice plan create \
  --name onepiece-plan \
  --resource-group infra-rg \
  --sku B1 \
  --is-linux
```
b. create webapp
```bash
az webapp create \
  --resource-group infra-rg \
  --plan onepiece-plan \
  --name onepiece-login-app \
  --runtime "JAVA|17-java17"
```
c. Configure app settings
```bash
az webapp config appsettings set \
  --name onepiece-login-app \
  --resource-group infra-rg \
  --settings \
  SPRING_DATASOURCE_URL="jdbc:sqlserver://onesqlserver123.database.windows.net:1433;database=onepiecedb;encrypt=true;trustServerCertificate=false;loginTimeout=30;" \
  SPRING_DATASOURCE_USERNAME="luckyadmin" \
  SPRING_DATASOURCE_PASSWORD="YourSecurePassword@123" \
  SPRING_DATASOURCE_DRIVER_CLASS_NAME="com.microsoft.sqlserver.jdbc.SQLServerDriver" \
  SPRING_JPA_HIBERNATE_DDL_AUTO=update \
  SPRING_JPA_SHOW_SQL=true \
  SPRING_JPA_PROPERTIES_HIBERNATE_DIALECT=org.hibernate.dialect.SQLServerDialect
```
15. Navigate to app.jar file which is target and change it to zip
```bash
zip app.zip app.jar
```
16. Deploy the zip file to azure app service
```bash
az webapp deploy \
  --resource-group infra-rg \
  --name onepiece-login-app \
  --src-path /target/app.zip \
  --type zip
```
<h2 align="center">Final Output</h2>

17. Browse the app service url you will see you application running and fill the **FORM** then click **set sail**. 

18. Navitage to your database and check the filled data stored in the database

https://github.com/user-attachments/assets/1807f254-0407-43c2-b0c9-3481b499e9f4

