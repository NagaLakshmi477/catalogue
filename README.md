# catalogue

create ROBOSHOP ---> floder --> create ---> save
ROBOSHOP --> create pipeline 

here versions are changing so we need read the version from package.json file
for this we need to install one pluggin inside json (pipeline utility steps) --> pluggin

Depending installations   

we need login into agent and run the 
dnf module disable nodejs
sudo dnf module enable nodejs:20 -y
dnf install nodejs -y

create a webhook ---> configure --> add --> github hook 

now we need to create docker image
docker build -t URL/catalogue:appVersion

now we need to push to ECR

AWS --> Amzon ECR --> private registry ---> repo --> create --> roboshop/catalogue --> mutable --> create
there is a instruction tab it will show all process
1. we need to login for this we need to crate credentails

pluggin --> aws credentails 

manage jenkins ---> credenatils ---> kind --> aws credentails --> aws-auth(id)--> ssh-cre --> acess key and secreat key 
pluggin --> aws steps
this for api hitting 


install docker plugin on jenkins agent

sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user

disconnect and connect agent 

run pipeline --> we can see image on ECR --> scan 

testing --> functional testing -->DEV --> devlopers/testers
            integration testing --> UAT --> here all components needs to communicate properly


now image is ready we need to deploy 

Create INFRA floder
-----------------
install terraform on node
Install required utilities

sudo dnf install -y dnf-utils yum-utils

Add HashiCorp repository

sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo

Install Terraform

sudo dnf install -y terraform

Verify installation

terraform -version

create node --00-vpc --> pipeline

configure -- 00-vpc/jenkins 

pluggin ---> ansi clour

If any pipeline fail after installing plugins we need to restart the jenkins
sudo systemctl restart jenkins


UPSTREAM and DOWN STREAM
======================

create pipeline for sg 
pipleline --10-sg
down copy from 00-vpc --> Same process 


catalogue --> For CI
catalogue-cd --> for CD
on node side we cam install kubectl
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.10/2026-04-08/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

cerate pipline for catalogue-ci and catalgue-cd on roboshop floder