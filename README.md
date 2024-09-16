# kops-kubernetes-cluster-configuration

# www.loreontechnologies.com 

# Setting up Kubernetes (K8s) Cluster on AWS Using KOPS

1.kops is a software use to create production ready k8s cluster in a cloud provider like AWS.

2. kOPS SUPPORTS MULTIPLE CLOUD PROVIDERS

3. Kops compete with managed Kubernetes services like EKS, AKS, and GKE

4. Kops is cheaper than the others.

5. Kops creates production-ready K8S.

6. KOPS creates resources like LoadBalancers, ASG, Launch Configuration, worker node Master node (CONTROL PLANE.

7. KOPS is IaaC

#!/bin/bash
# 1) Create Ubuntu EC2 instance in AWS

## 2a) create kops user
``` sh
 sudo adduser kops
 sudo echo "kops  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/kops
 sudo su - kops
 ```
 ##  2a) install AWSCLI
  ```sh
 sudo apt install awscli -y 
 #or
 sudo apt update -y
 sudo apt install unzip wget -y
 sudo curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
 sudo apt install unzip python -y
 sudo unzip awscli-bundle.zip
 sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
 ```
# 3) Install kops software on ubuntu instance:

 	#Install wget if not installed
 	sudo apt install wget -y
 	sudo wget https://github.com/kubernetes/kops/releases/download/v1.22.0/kops-linux-amd64
 	sudo chmod +x kops-linux-amd64
 	sudo mv kops-linux-amd64 /usr/local/bin/kops
#or
 ```
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

# 4) Install kubectl
```
curl -Lo kubectl https://dl.k8s.io/release/$(curl -s -L https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
# 5) Create an IAM role from AWS Console or CLI with below Policies.

	AmazonEC2FullAccess
	AmazonRoute53FullAccess
	AmazonS3FullAccess
	IAMFullAccess
	AmazonVPCFullAccess
	AmazonSQSFullAccess
	AmazonEventBridgeFullAccess



Then Attach IAM role to ubuntu server from Console Select KOPS Server --> Actions --> Instance Settings --> Attach/Replace IAM Role --> Select the role which
You Created. --> Save.



# 6) create an S3 bucket
# Execute the commands below in your KOPS control Server. use unique s3 bucket name. If you get bucket name exists error.

	aws s3 mb s3://opschool-kopsv
	aws s3 ls
	
    ex: s3://opschool-kopsv
     
	Expose environment variable:

    # Add env variables in bashrc
    vi .bashrc
	
	# Give Unique Name And S3 Bucket which you created.
	export NAME=opschool.k8s.local
	export KOPS_STATE_STORE=s3://opschool-kopsv
 
    source .bashrc
	
# 7) Create sshkeys before creating cluster

    ssh-keygen
 

# 8) Create kubernetes cluster definitions on S3 bucket

	kops create cluster --zones us-east-2c --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}
	
	kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub

# 9) Create kubernetes cluser

	 kops update cluster ${NAME} --yes

# 10) Validate your cluster(KOPS will take some time to create cluster ,Execute below commond after 3 or 4 mins)

	   kops validate cluster
	   
	   Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.class.k8s.local
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
 * read about installing addons at: https://kops.sigs.k8s.io/operations/addons.


# 11) connect to the master node
    sh -i ~/.ssh/id_rsa ubuntu@ipAddress
    
## 11b.  Enable PasswordAuthentication in the master server and assign passwd
```sh
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart
sudo passwd ubuntu
```

# 11c) To list nodes

	  kubectl get nodes 
 
# 12) To Delete Cluster

   kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes  
   
====================================================================================================


13 # IF you want to SSH to Kubernetes Master or Nodes Created by KOPS. You can SSH From KOPS_Server

sh -i ~/.ssh/id_rsa ubuntu@ipAddress
ssh -i ~/.ssh/id_rsa ubuntu@3.90.203.23
  
``

