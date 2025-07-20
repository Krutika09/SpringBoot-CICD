# Jenkins, Docker, and SonarQube Setup Guide

## 1. Allow Custom TCP 8080 and All Traffic
Make sure your cloud security group or local firewall allows:
- TCP port `8080`
- All outbound traffic

---

## 2. Install Java and Jenkins

```bash
sudo apt update -y
sudo apt install -y openjdk-17-jdk
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins
````

---

## 3. Docker Setup and Permissions

```bash
sudo apt install -y docker.io
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
```

---

## 4. Install and Configure SonarQube

```bash
sudo apt update && sudo apt install unzip -y

# Add user
sudo adduser sonarqube
sudo su - sonarqube

# Download and unzip
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip sonarqube-10.4.1.88267.zip
cd sonarqube-10.4.1.88267

# Set permissions
chmod -R 775 /home/sonarqube/sonarqube-10.4.1.88267
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-10.4.1.88267

# Start SonarQube
cd bin/linux-x86-64
./sonar.sh start
```

Access SonarQube:

```
http://<public-ip>:9000
```

Login using:

* **Username:** admin
* **Password:** admin

Then generate a secret token:

```
squ_ffff6c62821a7f22fed8ae76cf4a44f1e55602d0
```

Add this token in Jenkins as a **Secret Text Credential**.

---

## 5. Restart Jenkins From UI
```
http://public-ip:8080/restart
```

## 6. Operator Lifecycle Manager (OLM) Installation (For Kubernetes)

**Start Minikube (Git Bash recommended):**

```bash
minikube start --memory=4096 --cpus=2
```

**Install OLM:**

```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.32.0/install.sh | bash -s v0.32.0
```

---

## 7. Add Jenkins Credentials

* **DockerHub:** Username + Personal Access Token
* **GitHub:** Username + PAT with repo scope
* **SonarQube Token:** Secret Text Credential using the token created above

---

## 8. Sample Jenkinsfile Stage for SonarQube

```groovy
stage('Static Code Analysis') {
    environment {
        SONAR_URL = "http://<public-ip>:9000"
    }
    steps {
        script {
            withSonarQubeEnv('SonarQube') {
                sh 'mvn sonar:sonar'
            }
        }
    }
}
```
Replace `<public-ip>` with your actual instance IP and `SonarQube` with the credential ID you set.


