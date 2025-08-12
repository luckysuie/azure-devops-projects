# SpringMon: Prometheus & Grafana Monitoring for Spring Boot Applications
- create an ubuntu vm with all ports open
0. Login via ssh

1. Install git
```bash
	sudo apt update
	sudo apt install git -y
```
2. Install tree for folder structure
```bash	
sudo apt update
sudo apt install tree
``` 
3. Install maven and configure the path for global use
```bash
sudo apt update
wget https://dlcdn.apache.org/maven/maven-3/3.8.9/binaries/apache-maven-3.8.9-bin.tar.gz    
tar -xvzf apache-maven-3.9.9-bin.tar.gz
ls
mv apache-maven-3.9.9 maven
cd maven
pwd
cd 
ls -a
vi .bashrc
 export MAVEN_HOME=/home/LakshmiNarayana/maven #in place of my name put your VM username
 export PATH=$MAVEN_HOME/bin:$PATH
source ~/.bashrc
sudo apt install openjdk-21-jdk -y
mvn -v
```

4. clone the java springboot repo
```bash
git clone https://github.com/spring-projects/spring-petclinic
```

5. Install Docker
```bash
sudo apt update
sudo apt install docker.* -y
```
6. create Prometheus file and give some yaml
```bash
mkdir -p ~/prometheus
nano ~/prometheus/prometheus.yml
```
Yaml file
--------
```bash
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "spring-boot-app"
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ["20.63.13.48:8080"]  #replace with your VMIP here
```

7. Run Prometheus as a container
```bash
sudo docker run -d \
  --name=prometheus \
  -p 9090:9090 \
  -v ~/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```
Test : http://4.206.127.224:9090
	
8. Run Grafana as a container
```bash
sudo docker run -d \
  --name=grafana \
  -p 3000:3000 \
  grafana/grafana
```
Test: http://4.206.127.224:9090

9. Editing required files in Java application
--------------
- Edit pom.xml file with below
```bash
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

- Edit application.properties file with below
```bash
management.endpoints.web.exposure.include=health,info,prometheus
management.endpoint.prometheus.enabled=true
```

10. Build the app using maven commands
```bash
./mvnw clean package
```

- Run the java springboot application
```bash
java -jar target/*.jar
```

Test : http://4.206.127.224:8080

11. Grafana Setup and JVM Dashboard:
--------------
- Browse the public ip of your vm with 3000 port
- login to to Grafana using 
    - username :admin
    - password :admin

- Navigate to connections>Data sources>select Prometheus
- Prometheus server URL : http://20.63.13.48:9090
- At the end click save and test
<img width="1913" height="949" alt="Screenshot 2025-08-12 061820" src="https://github.com/user-attachments/assets/7692ba08-1905-40c6-b77f-a20d5e9ca4f7" />

- Navigate to Dashboards>top right corner>new>import
- Type 4701 in the ID and click load
- It will move you to another page like below then select your datasource in the promethesus dropdown
   - click on Import
  
<img width="1735" height="944" alt="Screenshot 2025-08-12 062205" src="https://github.com/user-attachments/assets/caaac923-117d-4da7-be53-4c21558544b7" />


Few Grafana Dashboard Id:
-------------------	
JVM (Micrometer) : Dashboard ID: 4701
Dashboard ID: 12464 — Spring Boot Statistics


Grafana Dashboards
-----------------------
1. JVM (Micrometer) Dashboard

<img width="1911" height="986" alt="Screenshot 2025-08-11 200356" src="https://github.com/user-attachments/assets/c499e0d5-336a-4bab-a44f-83895c59b773" />

2. Spring Boot Statistics
Screenshot


Promethesus Queries for the application
--------------
1. Total user and system CPU time spent in seconds.
<img width="1895" height="847" alt="Screenshot 2025-08-12 072754" src="https://github.com/user-attachments/assets/0b3d3e0e-298c-4dbc-af05-64765ad189ec" />


2. System cpu usage when application is running

<img width="1905" height="902" alt="Screenshot 2025-08-12 072709" src="https://github.com/user-attachments/assets/add62927-f3f0-47c9-9a64-5ea2b4d27931" />
