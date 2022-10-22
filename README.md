 # Setup Kubernetes Cluster on EC2 Instance Using Kops

•	Setup Kubernetes Cluster on EC2 Instance Using Kops
o	Step 1 : Create an EC2 Instance
o	Step 2:  Install AWSCLI
o	Step 3: Install Kubectl
o	Step 4: Create an IAM user with Route53, EC2, IAM and S3 full access
o	Step 5: Attach IAM user to ubuntu server
o	Step 6: Install Kops
o	Step 7: Create a Route53 private hosted zone
o	Step 8: Create S3 Bucket
o	Step 9: Create SSH Keys
o	Step 10: Create Kubernetes Cluster Definitions on S3 bucket
o	Step 11: Create Cluster

Step 2:  Install AWSCLI
We need to install and configure AWS CI on the instance to execute AWS commands. It will be required later on to interact with the Kubernetes cluster.

a) First, become root 'sudo su –'
# sudo yum update

b) To  download the awscli zip file.
# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

c) To unzip the awscli file.
# unzip awscliv2.zip

d) To install the awscli.
# sudo ./aws/install
Once this is done, you should be able to check the version of aws cli installed on the instance.

# aws --version
aws-cli/2.2.26 Python/3.8.8 Linux/5.4.0-1045-aws exe/x86_64.ubuntu.20 prompt/off
 
Step 3: Install Kubectl
Kubectl utility is used to interact with control plane in the Kubernetes cluster. It allows us to use CLI commands to do so. Download the file from below path.

# curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

Assign execution permission to the file.
# chmod +x ./kubectl

Move the file to the specified location.
# sudo mv ./kubectl /usr/local/bin/kubectl

Step 4: Create an IAM user with Route53, EC2, IAM and S3 full access
We can also create IAM role in place of IAM user. EC2 instance will require to access above services while installing Kubernetes and setting up the cluster.

•	Go to IAM -> Users
•	Click on "Add users" and give user a name. In the next step give user permission and also create a tag as shown below.
•	Once done click on "Create user"
•	Download the .csv file that contains Access key and Secret Key.
Step 5: Attach IAM user to AWS server
Now that we have created the IAM user with all needed permission, we will attach this user to the EC2 instance by following commands.

# aws configure
AWS Access Key ID [****************O76J]:
AWS Secret Access Key [****************bI6i]:
Default region name [us-east-2]: us-west-1
Default output format [None]: json

Step 6: Install Kops
Kops is a pre-requisite which is needed to create and manage AWS cluster. Use below curl command to download Kops utility from GitHub. More on Kops utility.

# curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64


# chmod +x kops-linux-amd64

Then move the executable file to /usr/local/bin location using below mv command.

# sudo mv kops-linux-amd64 /usr/local/bin/kops

Step 7: Create a Route53 private hosted zone
Under services in AWS console go to Route53 -> DNS Management -> Create hosted zone. Give it a name, select VPC and type as shown below. Once done click on "Create hosted zone".
Then provide Region and VPC ID as shown below.
Step 8: Create S3 Bucket
All cluster information will get stored in the created bucket. Use below command to create the bucket.

# aws s3 mb s3://k8s.bucket.cluster
make_bucket: k8s.bucket.cluster

Where,
mb -> make bucket
k8s.bucket.cluster -> name of the bucket
Once created, go to Amazon S3 service and validate if you can see the bucket created there as shown below.

# export KOPS_STATE_STORE=s3://k8s.bucket.cluster

Step 9: Create SSH Keys
Generate the SSH Key pair to enable key based authentication for our Kubernetes cluster.

# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:/etXqMbmDm0d93jceA7ZqIwyjW61nKKoiMehVrty/jg ubuntu@ip-172-31-24-136
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|                 |
|                 |
|         .       |
|        S .   ...|
|  ..      .o ..X+|
| o...    =.++.B.B|
|o+oE..  * ===o.= |
|+.===..+.+.*B.  .|
+----[SHA256]-----+
 
Step 10: Create Kubernetes Cluster Definitions on S3 bucket
Now you can create Kubernetes Cluster definitions on S3 bucket using below kops create cluster command.

# kops create cluster --cloud=aws --zones=us-east-2b --name=k8s.bucket.cluster --dns-zone=private-zone --dns private --state s3://k8s.bucket.cluster
 
Step 11: Create Cluster
Finally to create the cluster use below kops update cluster command.

# kops update cluster k8s.bucket.cluster --state s3://k8s.bucket.cluster –yes
Once above command gets executed, you will see master and worker nodes created. Verify them under EC2 service as shown below.
