Here is your updated and **well-formatted `README.md` file** with the new RDS and Kubernetes setup steps, fully cleaned and structured for GitHub:

---

````markdown
# Microservices Deployment Guide with Jenkins, Docker, RDS, and Kubernetes

## GitHub Repositories

- [Eureka Server](https://github.com/javaexpresschannel/eureka-server)  
- [User Service](https://github.com/javaexpresschannel/user-service)

## Webhooks

- GitHub Webhook: [http://3.71.34.65:8080/github-webhook/](http://3.71.34.65:8080/github-webhook/)

## Jenkins Installation

- [Jenkins Official Documentation](https://www.jenkins.io/doc/book/installing/linux/)

## Zipkin Configuration

- [Kubernetes Configurations](https://github.com/javaexpresschannel/kubernetes-configurations)

---

## Step-by-Step Microservices Deployment

1. Create an AWS account.  
2. Verify billing and add a payment method.  
3. Download and install [MobaXterm](https://mobaxterm.mobatek.net/download.html).  
4. Launch an EC2 instance and download the `.pem` key.  
5. Connect to EC2 using MobaXterm (SSH).  
6. Install Java  
7. Install Jenkins  
8. Install Docker  
9. Install Kubernetes  
10. Create RDS for `user-service`

---

## Java Installation

```bash
sudo apt-get update  
sudo apt install openjdk-17-jre-headless -y  
````

---

## Jenkins Installation

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update  
sudo apt-get install jenkins  
```

Access Jenkins via:
`http://<EC2-IP>:8080/`

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## RDS Creation for `user-service`

### Install AWS CLI

```bash
sudo su - jenkins
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### Configure AWS CLI

```bash
aws configure
# Enter:
# AWS Access Key: XXXX
# AWS Secret Key: XXXX
# Region: ap-south-1
```

### RDS Setup (MySQL)

```bash
aws rds create-db-instance \
  --db-instance-identifier userdb \
  --db-name ecomuserms \
  --db-instance-class db.t4g.micro \
  --engine mysql \
  --engine-version 8.0.40 \
  --master-username root \
  --master-user-password javaexpress1122 \
  --allocated-storage 20 \
  --storage-type gp2 \
  --publicly-accessible \
  --vpc-security-group-ids $(aws ec2 describe-security-groups \
    --filters Name=group-name,Values=default \
    --query "SecurityGroups[0].GroupId" --output text) \
  --backup-retention-period 0 \
  --no-multi-az \
  --region ap-south-1 \
  --no-deletion-protection
```

✅ After creation, **enable access in the Security Group manually**.

### RDS Credentials

* **Username**: `root`
* **Password**: `javaexpress1122`
* **Database Name**: `ecomuserms`
* **Connection String**: *(Dynamic - provided after RDS creation)*

### Delete RDS Instance (if needed)

```bash
aws rds delete-db-instance \
  --db-instance-identifier userdb \
  --skip-final-snapshot \
  --region ap-south-1
```

---

## Kubernetes Installation (EKS)

### AWS CLI (if not installed)

```bash
sudo su - jenkins
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

```bash
aws configure
# Set region: ap-south-1
```

### Install `kubectl`

```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
```

### Install `eksctl`

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

## Create EKS Kubernetes Cluster (⏳ \~20 mins)

```bash
eksctl create cluster \
  --name javaexpress-new-cluster1 \
  --version 1.29 \
  --region ap-south-1 \
  --nodegroup-name worker-nodes \
  --node-type t3.medium \
  --nodes 3
```

---

## Delete LoadBalancer Service

```bash
kubectl delete svc user-service-api
```

---

## Delete Kubernetes Cluster

```bash
eksctl delete cluster --name javaexpress-new-cluster1
```

---

## Final Note

✅ After verifying the Kubernetes setup, apply the Zipkin configuration files from the repository:

🔗 [https://github.com/javaexpresschannel/kubernetes-configurations](https://github.com/javaexpresschannel/kubernetes-configurations)

```

