
Github Link 
https://github.com/javaexpresschannel/eureka-server
https://github.com/javaexpresschannel/user-service

Webhooks
http://3.71.34.65:8080/github-webhook/

Jenkins Installation Steps 
https://www.jenkins.io/doc/book/installing/linux/

Zipkin Configuration 
https://github.com/javaexpresschannel/kubernetes-configurations

Step by Step for Microservices Deployment
------------------------------------------
1. Create an AWS account from the official website.  
2. Verify billing details and add a payment method.  
3. Download and install MobaXterm on your system.  
4. Launch an EC2 instance and download the .pem key.  
5. Connect to the EC2 instance using MobaXterm via SSH.
6. Install Java 
7. Install Jenkins
8. Install Docker 
9. Install Kubernetes


How to create EC2 Instance in AWS (ap-south-1)
---------------------------------
1) EC2 Creation in ubuntu (22.04)
2) Instance Type : t2.large (best for installation multiple softwares)
3) Create New Key Pair 
5) Enable one public ip 
6) Choose New Security Group  (jenkins_nsg)
	-> Type : All TCP & anywhere
7) Configure Storage 25	
7) Launch instance

How to connect in Mobaxterm?
---------------------------------
1) Download Software
	https://mobaxterm.mobatek.net/download.html
2) Open Software
3) Click Session -> SSH
	Remote Host : Public Ipv4 address
	Username : Default username (ubuntu)
	Private Key : download keypair in .pem extension
4) Launch and Click Accept
5) you can able to see EC2 Instance Server
6) Settings -> Configuration -> SSH -> SSH keepalive


Java Installation
----------------- 
sudo apt-get update
sudo apt install openjdk-17-jre-headless -y

Jenkins Installation
--------------------
Documentation Link:
https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

Open EC2 Instance IP Address and append 8080
	http://13.235.33.160:8080/

sudo cat /var/lib/jenkins/secrets/initialAdminPassword


Optional Commands:
------------------
sudo systemctl enable jenkins
sudo systemctl status jenkins
sudo systemctl stop jenkins
sudo systemctl start jenkins
-------------------

Setup Maven in Jenkins 
----------------------
For setting up the Maven Goto -> Manage Jenkins ->Tools-> Maven

Name : MAVE  
Version: 3.9.9

Admin Privileges for jenkins users 

sudo vi /etc/sudoers 
jenkins ALL=(ALL) NOPASSWD: ALL 
To Save the file 
	Enter i in keyword 
	Paste below command 
		jenkins ALL=(ALL) NOPASSWD: ALL 
	Esc
	:wq! (enter)
	
sudo cat -n /etc/sudoers (verification)
sudo su - jenkins

Docker Installation
-------------------
sudo apt install docker.io -y
sudo usermod -aG docker jenkins 
sudo chmod 666 /var/run/docker.sock

-------------if Restart-----------------------------------
sudo su - jenkins 
docker images

Docker Commands
---------------
docker build -t javaexpress/dockerimageName:latest . 
docker login -u username -p password 
docker push javaexpress/dockerimageName:latest 
------------------------------------------------------------------------

setup webhook in git repo 
-------------------------
https://github.com/javaexpresschannel/test1

1) Click settings on git repo 
2) Left side -> add webhook
3) PayloadUrl
http://3.71.34.65:8080/github-webhook/
4) add web hook 

Add docker credentails in jenkins 
---------------------------------
1) Manage Jenkins -> Credentails
2) Click on System 
3) Click on Global Credentails 
4) Add Credentails 
5) Choose Secret Text 
6) Secret : docker pwd 
	ID and Description : DOCKER_HUB_PASSWORD


How to bind password for docker in pipleine syntax
-------------------------------------------------
1) Choose withCredentails: Bind Credentails to variables 
2) Choose secret Text 
3) Variable Name : PASSWORD


Note:
Create two Jenkins jobs: one for `user-service` and another for `eureka-server`, then verify whether the corresponding Docker images are successfully built and available.


Before Running Kubernetes Create RDS Instance for user-service 
--------------------------------------------------------------

RDS - Creation for USER MS
--------------------------
Username : root 
Password : javaexpress1122
DB Name : ecomuserms
Connection String : Dynamic

aws rds create-db-instance --db-instance-identifier userdb --db-name ecomuserms --db-instance-class db.t4g.micro --engine mysql --engine-version 8.0.40 --master-username root --master-user-password javaexpress1122 --allocated-storage 20 --storage-type gp2 --publicly-accessible --vpc-security-group-ids $(aws ec2 describe-security-groups --filters Name=group-name,Values=default --query "SecurityGroups[0].GroupId" --output text) --backup-retention-period 0 --no-multi-az --region ap-south-1 --no-deletion-protection

Enable Security Group Manually 

Deletion 
--------
aws rds delete-db-instance \
  --db-instance-identifier userdb \
  --skip-final-snapshot \
  --region ap-south-1
  
Kubernetes Installation
-----------------------

AWSCLI
------
sudo su -  jenkins
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws –-version

aws configure 
	XXXX
	XXXX
	ap-south-1

Kubectl
------
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl 
sudo mv ./kubectl /usr/local/bin

EKSCTL
------
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin 
eksctl version

Create Kubernetes Cluster (20 min)
-------------------------
eksctl create cluster --name javaexpress-new-cluster1 --version 1.29 --region ap-south-1 --nodegroup-name worker-nodes --node-type t3.medium --nodes 3

Delete LoadBalancer
-------------------
kubectl delete svc springboot-svc 

Delete Kubernetes Cluster 
-------------------------
eksctl delete cluster --name javaexpress-new-cluster1

Note:
After verifying the Kubernetes setup, run the Zipkin configuration file from the following repository:  
https://github.com/javaexpresschannel/kubernetes-configurations








