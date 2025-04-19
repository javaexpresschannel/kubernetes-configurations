Kubernetes Setup in AWS and RDS Creation
----------------------------------------

AWSCLI
------
sudo apt-get update
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws –-version

aws configure 
	XXXXX
	XXXXX
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
eksctl create cluster --name javaexpress-new-cluster --version 1.29 --region ap-south-1 --nodegroup-name worker-nodes --node-type t3.medium --nodes 3

Delete LoadBalancer
-------------------
kubectl delete svc springboot-svc 

Delete Kubernetes Cluster 
-------------------------
eksctl delete cluster --name javaexpress-new-cluster


RDS - Creation for USER MS
--------------------------
aws rds create-db-instance --db-instance-identifier userdb --db-name ecomuserms --db-instance-class db.t4g.micro --engine mysql --engine-version 8.0.40 --master-username root --master-user-password javaexpress1122 --allocated-storage 20 --storage-type gp2 --publicly-accessible --vpc-security-group-ids $(aws ec2 describe-security-groups --filters Name=group-name,Values=default --query "SecurityGroups[0].GroupId" --output text) --backup-retention-period 0 --no-multi-az --region ap-south-1 --no-deletion-protection


Deletion 
--------
aws rds delete-db-instance \
  --db-instance-identifier userdb \
  --skip-final-snapshot \
  --region ap-south-1
