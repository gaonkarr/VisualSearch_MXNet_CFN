         ___        ______    
        / \ \      / / ___|   
       / _ \ \ /\ / /\___ \  
      / ___ \ V  V /  ___) | 
     /_/   \_\_/\_/  |____/   
 ----------------------------------------------------------------- 

This project creates 2 CloudFormation stacks :
1. **public-private-vpc.json**   
This stack deploys a Fargate cluster that is in a VPC with both\npublic and private subnets. Containers can be deployed into either\nthe public subnets or the private subnets, and there are two load\nbalancers. One is inside the public subnet, which can be used to\nsend traffic to the containers in the private subnet, and one in\nthe private subnet, which can be used for private internal traffic\nbetween internal services.


2. **private-subnet-public-loadbalancer.json**   
Deploy a service on AWS Fargate, hosted in a private subnet, but accessible via a public load balancer. 
This also launches an EC2 instance using Deep Learning Ubuntu AMI in the Public Subnet [not depicted in image]
The Deep Learning instance creates container image using Dockerfile mentioned in this repo. It pushes it to ECR and configures task definition.
The stack then creates a ECS cluster with the task definition.

![private subnet public load balancer](images/private-task-public-loadbalancer.png)

### Prerequisites

If you are using CLI to interact with AWS, then ensure CLI is configured. [Default region US-EAST-1, considering availability of used resources]

Ensure that you have a KeyPair available to access the Deep Learning Instance.

## Steps to run VisualSearch_MXNet Workshop

 
 
### Create CloudFormation Stacks :  
1. Launch the CloudFormation templates in order described above.
 
To launch using CLI :  
```
aws cloudformation create-stack --stack-name <<Stack1Name>> --template-body file:///templates/public-private-vpc.json --capabilities  CAPABILITY_IAM 
```

In the parameters/private-subnet-public-loadbalancer-params.json. Parameter Key : StackName ; ensure to update the Value provided in <<Stack1Name>>

```
aws cloudformation create-stack --stack-name <<Stack2Name>> --template-body file:///templates/private-subnet-public-loadbalancer.json --capabilities  CAPABILITY_IAM --parameters file:///parameters/private-subnet-public-loadbalancer-params.json 
```



2. Use the Outputs section to determine the Deep Learning EC2 Instance Public IP or simply look for EC2 instance with Tags : 
[{"Key": "Name","Value": "DeepLearningInstance"}, {"Key": "Project","Value": "VisualSearch_MXNetWorkshop"}]



3. Log into the Deep Learning instance using KeyPair provided during CloudFormation stack launch. 
Following command for MACs. 
More info AWS Documentation for information on [Configure the Client to Connect to the Jupyter Server](https://docs.aws.amazon.com/dlami/latest/devguide/setup-jupyter-configure-client.html) 


```
ssh -i ./<<keypair-name>>.pem -L 8888:127.0.0.1:8888 ubuntu@<<InstanceIP>>
```



### Visual Search with MXNet Gluon and HNSW

4. After logging in run following command to download the git repo :
 
```
git clone https://github.com/ThomasDelteil/VisualSearch_MXNet.git
```



5. Once the Jupyter Notebook is started, access it via your local browser using localhost link. The token is displayed in the terminal window where you launched the server. Look for something like:
```
Copy/paste this URL into your browser when you connect for the first time, to login with a token: http://localhost:8888/?token=0d3f35c9e404882eaaca6e15efdccbcd9a977fee4a8bc083
```

Copy the link and use that to access your Jupyter notebook server.
  


6. Goto https://github.com/ThomasDelteil/VisualSearch_MXNet and run the steps in Jupyter Notebook to create models and indexes.



### Steps to update the docker image and push it to Fargate

7. Once the model is created go back to the Deep Learning Instance terminal, run following commands to update the docker image and push it to ECR.
    

   7.1 Get docker login & build the Docker image using Dockerfile provided in "VisualSearch_MXNet/mms" folder.

```
    cd <path/to/project>/VisualSearch_MXNet/mms
    `aws ecr get-login --region us-east-1`
    docker build -t <repository-name>:latest .
```


  7.2 Check the image using command :

```
    docker images
```


   7.3 Tag the image with latest tag *[Check out 2nd CloudFormation stack outputs section for repository URI]*
```
    docker tag <repository-name>:latest <account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:latest
```


   7.4 Push the docker image to ECR repository

```
    docker push <account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:latest
```


   7.5 Update the service. Run following on local machine or Deep Learning Instance. *[Service name is in outputs section of 2nd CloudFormation stack.]*
```
    aws ecs update-service --service <ecs-service-name> --force-new-deployment
```



8. Open the 1st CloudFormation stack's output section, click on the link for "ExternalUrl". The browser should display...
