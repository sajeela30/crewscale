AWS ACCOUNT SETUP:
1.IAM USER:
Go to Services > IAM > Groups and create a new group that has the following access rights:
a)AmazonEC2FullAccess
b)IAMFullAccess
c)AmazonS3FullAccess

2.Now create a new user within this group
On your own machine, create the file ~/.aws/credentials with that users access key and this structure:

[default]
aws_access_key_id=xxxxx
aws_secret_access_key=xxxxx

3.Keypair:
create a new keypair on your own machine by running ssh-keygen -t rsa in a shell.

Note the path to the private key and replace the terraform variable privateKeyPath in modules/Infrastructure/main.tf with your own.

Click on Key Pairs in your EC2 Dashboard and import your public key.

4.Local Machine Setup:
Install the following on your machine :
use any tool for automating

terraform     (0.10.5)
Git bash
ansible       (2.3.2.0)
python2       (2.7.13)
pip2          (9.0.1)
pip2 install -U boto six ansible

5.Running code:
All you need to do to push to Github from local:


  command :git push origin master
Now one could "clone" that repository on another computer and not just get the latest code but the complete revision history on another computer.
6.Setting up 
Assuming your project is in a folder named "script" on your Desktop.
cd ~/Desktop/script
git init
git checkout -b develop
touch README.md

Add your remote.

  git remote add origin {{the link you just copied}}
explanation:

git :: The git command
remote add :: We're adding a remote connection for this repository
origin :: We're naming the remote 

7.Cloning an existing repository.
copy ssh url from github
cd ~/Desktop
  git clone {{the link you just copied}} script
  This creates a directory named "Project", clones the repository there and adds a remote named "origin" back to the source.

  cd Script
  git checkout develop
  
  If that last command fails

  git checkout -b develop
  
  6.Creating developmentcycle
  Create a feature branch as normal.
  git checkout develop
	git checkout -b my-feature-branch
 
 
 7.PreRequisites:
 AMI ID Parameter:
 Navigate to AWS Systems Manager -> Parameter store -> Create parameter.
 aws ssm get-parameters --names /aws/service/ecs/optimized-ami/amazon-linux/recommended --region us-east-1 --query "Parameters[].Value" --output text | jq .
 
 
 8.ECS Cluster Instance Profile:
  Create new EcsInstanceRole IAM role, select AWS Service -> EC2 as Trusted Entity. Attach following permissions policies:

AmazonEC2RoleforSSM
AmazonEC2ContainerServiceforEC2Role
AWSLambdaBasicExecutionRole

9.CloudFormation Parameter File:
The file called blank_paramter_file.json in the project. You can copy this file to something new and with a more meaninigful name. eg: dev-cluster.json then fill out the parameters.

example file:
[
  {
    "ParameterKey": "EcsClusterName",
    "ParameterValue": ""
  }, 
  {
    "ParameterKey": "EcsAmiParameterKey",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "IamRoleInstanceProfile",
    "ParameterValue": ""
  }, 
  {
    "ParameterKey": "EcsInstanceType",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "EbsVolumeSize",
    "ParameterValue": ""
  }, 
  {
    "ParameterKey": "ClusterSize",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "ClusterMaxSize",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "KeyName",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "SubnetIds",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "SecurityGroupIds",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "DeploymentS3Bucket",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "LifecycleLaunchFunctionZip",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "LifecycleTerminateFunctionZip",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "LambdaFunctionRole",
    "ParameterValue": ""
  }
]
...................................................................


Parameter description:

a)EcsClusterName
b)EcsAmiParameterKey: is the EC2 Parameter store that contains the AMI ID 
c)IamRoleInstanceProfile: This is the Name of the EC2 Instance profile used by the ECS cluster members. 
d)EcsInstanceType: Is the instance type to use for the cluster.
e)EbsVolumeSize: Is the size of the Docker storage setup that is created using LVM. ECS typically defaults to 100GB.
f)ClusterSize: Is the desired number of EC2 Instances for the cluster.
g)ClusterMaxSize: This value should always be double the amount contained in ClusterSize. CloudFormation has no 'math' operators or I wouldn't prompt for this. This allows rolling updates to be performed safely by doubling the cluster size then contracting back.
h)KeyName: Name of the EC2 keypair to place on the ECS Instance to support SSH.
i)SubnetIds:  eg: subnet-a70508df,subnet-e009eb89.
j)SecurityGroupIds:  eg: sg-bd9d1bd4,sg-ac9127dca (a single value is fine).
k)DeploymentS3Bucket: This is the bucket where our two Lambda Functions for scaleup/scaledown lifecycle hooks can be found.
l)LifecycleLaunchFunctionZip:  where the ecs-lifecycle-hook-launch.zip contents can be found.
m)LifecycleTerminateFunctionZip: ecs-lifecycle-hook-terminate.zip contents can be found.



Parameter Deployment file:
[
  {
    "ParameterKey": "EcsClusterName",
    "ParameterValue": "DevCluster"
  }, 
  {
    "ParameterKey": "EcsAmiParameterKey",
    "ParameterValue": "/ami/ecs/latest"
  },
  {
    "ParameterKey": "IamRoleInstanceProfile",
    "ParameterValue": "EcsInstanceRole"
  }, 
  {
    "ParameterKey": "EcsInstanceType",
    "ParameterValue": "m4.large"
  },
  {
    "ParameterKey": "EbsVolumeSize",
    "ParameterValue": "100"
  }, 
  {
    "ParameterKey": "ClusterSize",
    "ParameterValue": "2"
  },
  {
    "ParameterKey": "ClusterMaxSize",
    "ParameterValue": "4"
  },
  {
    "ParameterKey": "KeyName",
    "ParameterValue": "dev-cluster"
  },
  {
    "ParameterKey": "SubnetIds",
    "ParameterValue": "subnet-a70508df,subnet-e009eb89"
  },
  {
    "ParameterKey": "SecurityGroupIds",
    "ParameterValue": "sg-bd9d1bd4"
  },
  {
    "ParameterKey": "DeploymentS3Bucket",
    "ParameterValue": "ecs-deployment"
  },
  {
    "ParameterKey": "LifecycleLaunchFunctionZip",
    "ParameterValue": "ecs-lifecycle-hook-launch.zip"
  },
  {
    "ParameterKey": "LifecycleTerminateFunctionZip",
    "ParameterValue": "ecs-lifecycle-hook-terminate.zip"
  },
  {
    "ParameterKey": "LambdaFunctionRole",
    "ParameterValue": "LambdaECSScalingRole"
  }
]

.......................................................................
10.Deploying the above parameter template:

An example deploying through the CLI. This assumes we have a stack named ecs-cluster.yaml and a parameter file named dev-cluster.json. It also uses the --profile argument to assure the CLI assumes a role in the right account for deployment.

aws cloudformation create-stack \
  --stack-name ecs-dev \
  --template-body file://./ecs-cluster.yaml \
  --parameters file://./dev-cluster.json \
  --region us-east-1 \
  --profile devAdmin
  
  
 Lifecycle updates (new AMI):
 aws cloudformation update-stack \
  --stack-name ecs-dev \
  --template-body file://./ecs-cluster.yaml \
  --parameters file://./dev-cluster.json \
  --region us-east-1 \
  --profile devAdmin
  
  
  
  (scale in/out):
  aws cloudformation update-stack \
  --stack-name ecs-dev \
  --template-body file://./ecs-cluster.yaml \
  --parameters file://./dev-cluster.json \
  --region us-east-1 \
  --profile devAdmin
  
  
  This could also be done via the CloudFormation web UI as desired!
  
  
  11.Checking the creations:
  Go to Aws console and check for 
             AWS account
             ECR
             ECS / Cluster
             EC2 instance
             Load balancer
             Proper IAMs
             Security groups
             VPN and subnet
             
             
  








 
  
  
