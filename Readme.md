# **🚀 Jenkins, Docker, SonarQube & Argo CD Setup Guide**

## ✅ 1. Create EC2 Instance

* **OS:** Ubuntu
* **Instance Type:** `t2.large`
* **Security Group:** Allow:

  * **TCP 8080** → For Jenkins
  * **TCP 9000** → For SonarQube
  * **All Outbound Traffic**

## ✅ 2.  Installation 

### Install Java, Jenkins & Maven

```bash
# Java & Maven Installation
sudo apt update -y
sudo apt install -y openjdk-17-jdk maven

# Jenkins Repository & Installation
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update -y
sudo apt-get install -y jenkins
```


### Install Docker & Assign Permissions

```bash
# Install Docker
sudo apt install -y docker.io

# Add Jenkins & Ubuntu to Docker Group
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu

# Restart Docker Service
sudo systemctl restart docker
```


### Install and Configure SonarQube 

```bash
# Prerequisites
sudo apt update && sudo apt install -y unzip

# Add SonarQube User
sudo adduser sonarqube
sudo su - sonarqube

# Download SonarQube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip sonarqube-10.4.1.88267.zip
cd sonarqube-10.4.1.88267

# Permissions
chmod -R 775 ~/sonarqube-10.4.1.88267
chown -R sonarqube:sonarqube ~/sonarqube-10.4.1.88267

# Start SonarQube
cd bin/linux-x86-64
./sonar.sh start
```

**🔗 Access Jenkins UI**:

`http://<public-ip>:8080` 
  * **Username**: `admin`
  * **Password**: Get Jenkins Password from `/var/lib/jenkins/secrets/initialAdminPassword`


**🔗 Access SonarQube UI**:
`http://<public-ip>:9000`
  * **Username**: `admin`
  * **Password**: `admin`

---

## ✅ 3. Credentials Setup

###  🔐 Jenkins
Go to: **Jenkins → Manage Jenkins → Credentials → (global) → Add Credentials**

### 🔐 SonarQube Token

* **Kind:** Secret text
* **Secret:** *Paste your SonarQube token*
* **ID:** `sonar-token`


### 🔐 GitHub Token

* **Kind:** Username with password
* **Username:** *Your GitHub username*
* **Password:** *GitHub Personal Access Token (PAT)*
* **ID:** `github-token`



### 🔐 DockerHub Credential

* **Kind:** Username with password
* **Username:** *Your DockerHub username*
* **Password:** *Your DockerHub password or token*
* **ID:** `dockerhub-cred`

### Restart Jenkins From UI
```
http://public-ip:8080/restart
```
---

## ✅ 4. Jenkins Pipeline Setup (Job Creation)

1. Go to: **Jenkins → New Item**

2. **Name** your job

3. Select: **Pipeline → OK**

4. In **Pipeline section**:

   * **Definition**: Pipeline script from SCM
   * **SCM**: Git
   * **Repository URL**: Your GitHub repo
   * **Branch**: `*/main`
   * **Script Path**: `spring-boot-app/Jenkinsfile` (or correct path)

5. **Save & Build**

---

## ✅ 5. Argo CD Setup on Minikube 

### 🧱 Start Minikube

```bash
minikube start --memory=4096 --cpus=2
```

### Operator Lifecycle Manager (OLM) Installation (For Kubernetes)

```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.32.0/install.sh | bash -s v0.32.0
```

```
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
```

```
kubectl get csv -n operators
```

### 📦 Apply ArgoCD Manifest

```bash
kubectl apply -f kube-argo-cd-basic.yaml
```

### 🔍 Check Pods

```bash
kubectl get pods -n argocd
```

### 🔄 Change Service Type (To NodePort)

```bash
kubectl edit svc argocd-server -n argocd
```

* Change `type: ClusterIP` to `type: NodePort`
* Save & exit

---

### 🌐 Get ArgoCD URL

```bash
minikube service argocd-server -n argocd
```

### 🔑 Get and Decode ArgoCD Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

Find the base64 encoded password and decode it:

```bash
echo 'cGFzc3dvcmQ=' | base64 --decode
```

---

### 🔓 Login to Argo CD

* **URL:** Use NodePort URL from `minikube service`
* **Username:** `admin`
* **Password:** *(decoded password)*
