
# Jenkins CI/CD with SonarQube, Docker, Minikube & Argo CD on Azure

This project demonstrates a **complete CI/CD pipeline on Azure** using:

* Jenkins (CI)
* SonarQube (Code Quality & Security)
* Docker (Containerization)
* Minikube (Kubernetes Cluster)
* Argo CD (GitOps CD)

---

## Pre-Requisites

* Azure Virtual Machine (Ubuntu)
* Network Security Group (NSG) access
* Internet connectivity
* `sudo` privileges
* GitHub & Docker Hub accounts

---

## Step 0: Create Azure Virtual Machine

Create an **Ubuntu VM** from the Azure Portal.

### Connect VM from Local System

```bash
ssh -i C:\Users\NITRO\OneDrive\Desktop\jenkins-argocd-gitopsKey.pem azureuser@40.81.224.134
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img2.jpeg)

---

## Step 1: Install Java (JDK)

Jenkins requires Java to run.

```bash
sudo apt update
sudo apt install openjdk-17-jre -y
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img3.jpeg)
### Verify Java Installation

```bash
java -version
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img4.jpeg)

---

## Step 2: Install Jenkins

### Add Jenkins Repository Key

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img5.jpeg)



### Add Jenkins Repository

```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img5.jpeg)

### Install Jenkins

```bash
sudo apt update
sudo apt install jenkins -y
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img5.jpeg)


### Start Jenkins Service

```bash
sudo systemctl start jenkins
sudo systemctl status jenkins
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img6.jpeg)

---

## Step 3: Configure Azure Network Security Group (NSG)

Jenkins runs on **port 8080**, which must be allowed.

### Allow Port 8080

1. Azure Portal → Virtual Machines
2. Select VM → Networking
3. Add **Inbound Port Rule**

**Configuration:**

* Source: Any / My IP
* Destination Port: `8080`
* Protocol: TCP
* Action: Allow
* Priority: `1000`
* Name: Jenkins

![alt text](java-maven-sonar-argocd-helm-k8s/img/img7.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img8.jpeg)


> ⚠️ **Security Best Practice:**
> Never allow all ports. Only allow required ports.

---

## Step 4: Access Jenkins

```text
http://<azure-vm-public-ip>:8080
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img9.jpeg)

---

## Step 5: Unlock Jenkins

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img10.jpeg)


* Paste password
* Click **Continue**


---

## Step 6: Install Jenkins Plugins

* Select **Install Suggested Plugins**
* Wait for installation


![alt text](java-maven-sonar-argocd-helm-k8s/img/img11.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img12.jpeg)



---

## Step 7: Create Admin User

* Create admin user (recommended)
* Or skip


---

## Jenkins Installed Successfully 🎉
create jenkins pipeline

![alt text](java-maven-sonar-argocd-helm-k8s/img/img13.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img14.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img15.jpeg)


---

## Step 8: Install Required Jenkins Plugins

### Docker Pipeline Plugin

1. Jenkins → Manage Jenkins → Manage Plugins
2. Available → Search **Docker Pipeline**
3. Install

![alt text](java-maven-sonar-argocd-helm-k8s/img/img16.jpeg)

### SonarQube Plugin

* Install **SonarQube Scanner** plugin

![alt text](java-maven-sonar-argocd-helm-k8s/img/img17.jpeg)

* Restart Jenkins


---

## Step 9: Install SonarQube on Azure VM

```bash
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip sonarqube-10.4.1.88267.zip
sudo mv sonarqube-10.4.1.88267 /opt/sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube
sudo chmod -R 775 /opt/sonarqube
cd /opt/sonarqube/bin/linux-x86-64
./sonar.sh start
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img18.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img20.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img21.jpeg)



---

## Allow SonarQube Port (9000)

Add NSG rule:

* Port: `9000`
* Priority: `1100`
* Name: SonarQube

![alt text](java-maven-sonar-argocd-helm-k8s/img/img22.jpeg)

### Access SonarQube

```text
http://<azure-vm-public-ip>:9000
```

* Username: `admin`
* Password: `admin`

![alt text](java-maven-sonar-argocd-helm-k8s/img/img23.jpeg)

---

## Generate SonarQube Token

* Generate token for Jenkins integration

```
sqa_xxxxxxxxxxxxxxxxx
```


### Add Token to Jenkins

* Jenkins → Manage Credentials
* Add **Secret Text**

![alt text](java-maven-sonar-argocd-helm-k8s/img/img25.jpeg)

---

## Step 10: Docker Agent Configuration

### Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img26.jpeg)

### Grant Docker Permissions

```bash
sudo su -
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img27.jpeg)


---

## Restart Jenkins


## ✅ CI Setup Completed

Jenkins + Docker + SonarQube configured successfully.

---

## Step 11: Create Kubernetes Cluster using Minikube

```bash
minikube start --memory=3000 --driver=<driver-name>
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img28.jpeg)

---

## Step 12: Install Argo CD using OperatorHub

GitOps tools are installed using **Kubernetes Operators**.

1. Go to **OperatorHub**
2. Search **Argo CD**
3. Follow installation commands

```bash
kubectl get pods -n operators
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img29.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img30.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img31.jpeg)




---

## Step 13: Jenkins Credentials Setup

Add credentials in Jenkins:

* GitHub credentials
* Docker Hub credentials

![alt text](java-maven-sonar-argocd-helm-k8s/img/img33.jpeg)
Then **build and run the Jenkins pipeline**.
![alt text](java-maven-sonar-argocd-helm-k8s/img/img32.jpeg)

---

## Step 14: Code Quality Check

* Jenkins triggers SonarQube scan
* View vulnerabilities & code quality in SonarQube UI
![alt text](java-maven-sonar-argocd-helm-k8s/img/img34.jpeg)

---

## Step 15: Create Argo CD Application

### Create Manifest

```bash
argocd-basic.yml
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img35.jpeg)

Apply:

```bash
kubectl apply -f argocd-basic.yml
```

Check pods:

```bash
kubectl get pods
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img37.jpeg)

---

## Expose Argo CD UI

```bash
kubectl edit svc example-argocd-server
```

Change:

```yaml
type: NodePort
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img39.jpeg)

Check services:

```bash
kubectl get svc
minikube service list
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img41.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img42.jpeg)


---

## Access Argo CD UI

* Username: `admin`
* Password: Retrieve from secret

```bash
kubectl get secret example-argocd-cluster -o yaml
```

![alt text](java-maven-sonar-argocd-helm-k8s/img/img44.jpeg)


*ArgoCD stores its admin password in Base64 format
*You must decode the Base64 value to get the actual password                                                     
*Run:
```bash
echo secret | base64 -d
```
*Copy the password and paste it in the place of password in ArgoCD UI

---

## Deploy Application using Argo CD

1. Click **Create Application**
2. Provide:

   * Repo URL
   * Path
   * Cluster destination
3. Create application

Argo CD will automatically:

* Deploy manifests
* Maintain desired state
![alt text](java-maven-sonar-argocd-helm-k8s/img/img46.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img47.jpeg)
![alt text](java-maven-sonar-argocd-helm-k8s/img/img48.jpeg)



---

## Verify Deployment

```bash
kubectl get deploy
kubectl get pods
```
![alt text](java-maven-sonar-argocd-helm-k8s/img/img49.jpeg)

---

## 🎉 CI/CD with GitOps Completed Successfully

Your application is now:

* Built via Jenkins
* Scanned via SonarQube
* Containerized using Docker
* Deployed using Argo CD
* Running on Kubernetes (Minikube)

---
