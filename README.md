Deploy Netflix Clone on AWS with Jenkins - DevSecOps Project

![image](https://github.com/user-attachments/assets/54d3aa7d-e9f3-4690-afd2-938abb841c7a)

Hello guys, we will be rolling out a Netflix clone. We will be using Jenkins as our CICD tool and deploying our application to a Docker container and Kubernetes Cluster and monitoring the Jenkins and Kubernetes metrics via Grafana, Prometheus and Node exporter. I Hope this informative blog proves useful.
Phase 1: Initial Setup and Deployment
Step 1: Launch EC2 (Ubuntu)
Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or reuse an old one. Put HTTP and HTTPS settings in Security Group and keep all ports open (not healthy to leave all ports open but for learning purpose it's alright).
•	Provision an EC2 instance on AWS with Ubuntu 22.04.
•	Connect to the instance using SSH.

Step 2: Clone the Repository
Update all system packages and clone the application repository:
sudo apt update && sudo apt upgrade -y
git clone https://github.com/N4si/DevSecOps-Project.git
cd DevSecOps-Project

 ![image](https://github.com/user-attachments/assets/ef7244a8-4b57-40a4-aad7-9bbb8666a1ab)



Step 3: Install Jenkins on EC2 instance: 

vi jenkins.sh #make sure run in Root (or) add at userdata while ec2 launch


#!/bin/bash
sudo apt update -y
#sudo apt upgrade -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins

![image](https://github.com/user-attachments/assets/7722f455-c6b1-4fab-8983-f941092b3166)



sudo chmod 777 jenkins.sh
./jenkins.sh    # this will installl jenkins



After Jenkins has been installed, you will have to go to your AWS EC2 Security Group and open Inbound Port 8080 since Jenkins runs on Port 8080.

Now, grab your Public IP Address

<EC2 Public IP Address:8080>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

 
Access Jenkins: http://<public-IP>:8080

Unlock Jenkins using an administrative password and install the suggested plugins.

 ![image](https://github.com/user-attachments/assets/5a5680c4-3e35-4596-8f37-189d4e281328)


Step 3.1: Install Jenkins, Docker and Trivy
Install Docker on the EC2 instance:

sudo apt install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock

Once the docker is installed successfully, I create a sonarqube contain(make sure to add 9000 inbound ports in the Security Group) 
![image](https://github.com/user-attachments/assets/b52679cb-a6ec-4409-9b42-188ec069e333)

 
Build and run the application using Docker:

docker build -t netflix .
docker run -d --name netflix -p 8081:80 netflix:latest
To stop and remove the container/image:
docker stop netflix
docker rm netflix
docker rmi -f Netflix
![image](https://github.com/user-attachments/assets/d473cb0a-5137-44dc-b88a-304fe2f67806)

 ![image](https://github.com/user-attachments/assets/6a71e79c-aed1-4c4e-be1d-dd8f3949fd71)

To install Trivy: 

sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy 

To scan Image using Trivy


 ![image](https://github.com/user-attachments/assets/40335c66-02da-4caa-a97d-eddafb498b50)

Note: You will encounter an error due to a missing API key.

Error : I got 
Firstly make sure stop and remove the server : I got an error without stopping server
![image](https://github.com/user-attachments/assets/34425120-a310-47b6-bd49-884c172a2cd8)
![image](https://github.com/user-attachments/assets/418f93d3-3e58-4fd3-b4a4-af54c4fab5a8)



Step 4: Get the TMDB API Key
1.	Navigate to TMDB and log in or create an account.
2.	Go to Profile → Settings → API.
3.	Click Create, accept terms, and generate your API key.
Rebuild the Docker image with your API key:
 ![image](https://github.com/user-attachments/assets/280b06c5-4767-4726-a604-e04f8acb7a43)

docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .

 ![image](https://github.com/user-attachments/assets/5bf7de5f-710e-491f-b803-edf72599fb52)




Phase 2: Security & Code Analysis

Install Trivy for vulnerability scanning:
sudo apt install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update && sudo apt install trivy –y

 ![image](https://github.com/user-attachments/assets/0d878906-dcd1-49ec-afe1-2276635da5f8)


 

 


Scan Docker images using Trivy:
trivy image netflix:latest

 


 





Install SonarQube and Trivy
Run SonarQube using Docker:
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

 ![image](https://github.com/user-attachments/assets/f371e5cf-3c77-4ae4-820d-ad8b17691a63)

Access SonarQube at: http://<public-IP>:9000 (Default login/pss: admin/admin).

Phase 3: CI/CD Setup with Jenkins
Install Jenkins on EC2
Install Java and Jenkins:
sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
java -version

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update && sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable Jenkins



 
 
 
Go to Manage Jenkins → Plugin Manager → Available Plugins, and install the following:
1.	Eclipse Temurin Installer
2.	SonarQube Scanner
3.	NodeJS Plugin
4.	Email Extension Plugin
5.	Docker, Docker Commons, Docker Pipeline, Docker API, Docker Build Step
6.	OWASP Dependency-Check
 
 
Configure Jenkins
1.	Go to Manage Jenkins → Global Tool Configuration
2.	Configure:
o	JDK 17
o	NodeJS 16
o	Sonar Scanner
 

3.	Add SonarQube Token:
o	Manage Jenkins → Credentials → Add Secret Text
o	Use this token in the pipeline.
Then after go to sonarqube>> administration?>>> security>>> user >> create token>> copy the token. 

Come to Jenkins>>> crediantial 

 
Come to manage Jenkins >> system 
 


Create CI/CD Pipeline in Jenkins:
Come to jenkin dashboard>>> new item>>pipeline

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}


And run the pipeline
 

 

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix ."
                       sh "docker tag netflix nasi101/netflix:latest"
                       sh "docker push nasi101/netflix:latest"
                    }
                }
            }
        }
        stage("Trivy Image Scan") {
            steps {
                sh "trivy image nasi101/netflix:latest > trivyimage.txt"
            }
        }
        stage('Deploy to Container') {
            steps {
                sh 'docker run -d --name netflix -p 8081:80 nasi101/netflix:latest'
            }
        }
    }
}



If you encounter a Docker login failure in Jenkins:
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
 



 
 

 


 

 

 
 

 

 

Phase 4: Monitoring with Prometheus & Grafana
Install Prometheus
Set up Prometheus and Grafana to monitor your application
Installing Prometheus:

First, create a dedicated Linux user for Prometheus and download Prometheus:
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-*.tar.gz
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo mv prometheus-2.47.1.linux-amd64/prometheus /usr/local/bin/
Create a Prometheus Service:
sudo nano /etc/systemd/system/prometheus.service

Add the following:
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus

[Install]
WantedBy=multi-user.target
Start and enable Prometheus:
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
![image](https://github.com/user-attachments/assets/36ea4e7d-ca23-45e6-96b7-9df5460d9d68)

 ![image](https://github.com/user-attachments/assets/703b0292-2199-4e9d-8382-fff632af0aea)
