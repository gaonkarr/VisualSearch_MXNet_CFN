         ___        ______    
        / \ \      / / ___|   
       / _ \ \ /\ / /\___ \  
      / ___ \ V  V /  ___) | 
     /_/   \_\_/\_/  |____/   
 ----------------------------------------------------------------- 

This project creates 2 CloudFormation stacks :
1. public-private-vpc
This stack deploys a Fargate cluster that is in a VPC with both\npublic and private subnets. Containers can be deployed into either\nthe public subnets or the private subnets, and there are two load\nbalancers. One is inside the public subnet, which can be used to\nsend traffic to the containers in the private subnet, and one in\nthe private subnet, which can be used for private internal traffic\nbetween internal services.
2. private-subnet-public-loadbalancer
Deploy a service on AWS Fargate, hosted in a private subnet, but accessible via a public load balancer. This also launches an EC2 instance using Deep Learning Ubuntu AMI
The stack creates a ECS cluster with task definition using the "awsdeeplearningteam/mms_cpu" container image.

To begin :
1. Launch the CloudFormation templates in order mentioned above. Before creating the stacks ensure that you have a KeyPair available to access the Deep Learning Instance.

To launch using CLI :
aws cloudformation create-stack --stack-name mystackMMS --template-body file:///templates/public-private-vpc.json --capabilities  CAPABILITY_IAM 

aws cloudformation create-stack --stack-name mystackMMS2 --template-body file:///templates/private-subnet-public-loadbalancer.json --capabilities  CAPABILITY_IAM --parameters file:///parameters/private-subnet-public-loadbalancer-params.json 


2. Use the Outputs section to determine the Deep Learning EC2 Instance Public IP or simply look for EC2 instance with Tags : 
[{"Key": "Name","Value": "DeepLearningInstance"}, {"Key": "Project","Value": "VisualSearch_MXNetWorkshop"}]

3. Log into the instance using KeyPair provided during CloudFormation stack launch. 
ssh -i ./<<keypair-name>>.pem -L 8888:127.0.0.1:8888 ubuntu@<<InstanceIP>>

4.  After logging in run following commands :

[[[[NEED HELP IN THIS SECTION]]]]]  

5.  Once the Jupyter Notebook is started, access it via your local browser using localhost link.
6.  Run the steps in Jupyter Notebook to create models and indexes.
7.  Once the model is created, run following commands to update the docker image and push it to ECR.
8.  

