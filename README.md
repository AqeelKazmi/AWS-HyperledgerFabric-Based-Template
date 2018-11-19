## AWS-HyperledgerFabric-Based-Template
This document is regarding setup of AWS Instance of Fabric based template.

 **_ Note: These steps assume that you have AWS account credentials._**
 
## Steps to create AWS Instance of fabric based template:

### 1. Create IAM user
-	Use your AWS account email address and password to sign in as the AWS account root user to the IAM console at https://console.aws.amazon.com/iam/.
-	In the navigation pane of the console, choose Users, and then choose Add user
-	For User name, type Administrator.
-	Select the check box next to AWS Management Console access, select Custom password, and then type the new user's password in the text   box. You can optionally select Require password reset to force the user to create a new password the next time the user signs in.
-	Choose Next: Permissions
-	On the Set permissions for user page, choose Add user to group. 
-	Choose Create group
-	In the Create group dialog box, type Administrators.
-	For Filter, choose Job function.
-	In the policy list, select the check box for AdministratorAccess. Then choose Create group.
-	Back in the list of groups, select the check box for your new group. Choose Refresh if necessary to see the group in the list. 
-	Choose Next: Review to see the list of group memberships to be added to the new user. When you are ready to proceed, choose Create       user. 


### 2. Create Key Pair
**_Note: Create the key pair in the same Region that you use to launch the Fabric network._**
-	Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/. 
-	From the navigation bar, select a Region for the key pair. You can select any Region that's available to you, regardless of your         location, but key pairs are specific to a Region. For example, if you plan to launch an instance in the US East (Ohio) region, you       must create a key pair for the instance in the same Region. 
-	In the navigation pane, choose Key Pairs, Create Key Pair. 
-	For Key pair name, enter a name for the new key pair. Choose a name that is easy for you to remember, such as your IAM user name,       followed by -key-pair, plus the region name. For example, me-key-pair-useast2. Choose Create. 
-	The private key file is automatically downloaded by your browser. The base file name is the name that you specified as the name of       your key pair, and the file name extension is .pem. Save the private key file in a safe place.

**_Important: This is the only chance for you to save the private key file. You provide the name of your key pair when you launch the      Fabric network._** 

### 3. Create a VPC and Subnets
**_Note: The configuration you specify in this tutorial creates an Application Load Balancer, which requires two public subnets in different Availability Zones. In addition, a private subnet is required for the container instances, and the subnet must be in the same Availability Zone as the Application Load Balancer. You first use the VPC Wizard to create one public subnet and a private subnet in the same Availability Zone. You then create a second public subnet within this VPC in a different Availability Zone._**
     **To create an Elastic IP address**
-	Open the Amazon VPC console at https://console.aws.amazon.com/vpc/. 
-	Choose Elastic IPs, Allocate new address, Allocate. 
-	Make a note of the Elastic IP address that you create and choose Close. 
-	In the list of Elastic IP addresses, find the Allocation ID for the Elastic IP address created earlier. You use this when you create     the VPC. 
     ** To create the VPC**
-	From the navigation bar, select a Region for the VPC. VPCs are specific to a Region, so select the same Region in which you created     your key pair in and where you are launching the Ethereum stack. For more information, see Create a Key Pair. 
-	On the VPC dashboard, choose Start VPC Wizard. 
-	On the Step 1: Select a VPC Configuration page, choose VPC with Public and Private Subnets, Select. 
-	On the Step 2: VPC with Public and Private Subnets page, leave IPv4 CIDR block and IPv6 CIDR block to their default values. For VPC     name, enter a friendly name. 
-	For Public subnet's IPv4 CIDR, leave the default value. For Availability Zone, choose a zone. For Public subnet name, enter a friendly   name. You specify this subnet as one of the first of two subnets for the Application Load Balancer when you use the template. Note the   Availability Zone of this subnet because you select the same Availability Zone for the private subnet, and a different one for the       other   public subnet. 
-	For Private subnet's IPv4 CIDR, leave the default value. For Availability Zone, select the same Availability Zone as in the previous     step. For Private subnet name, enter a friendly name. 
-	For Elastic IP Allocation ID, select the Elastic IP address that you created earlier. 
-	Leave the default values for other settings.
-	Choose Create VPC. 


###To create the second public subnet in a different Availability Zone
-	Choose Subnets and then select the public subnet that you created earlier from the list. Select the Route Table tab and note the Route   table ID. You specify this same route table for the second public subnet below. 
-	Choose Create Subnet. 
-	For Name tag, enter a name for the subnet. You use this name later when you create the bastion host in this network. 
-	For VPC, select the VPC that you created earlier. 
-	For Availability Zone, select a different zone from the zone that you selected for the first public subnet. 
-	For IPv4 CIDR block, enter 10.0.2.0/24. 
-	Choose Yes, Create. The subnet is added to the list of subnets. 
-	With the subnet selected from the list, choose Subnet Actions, Modify auto-assign IP settings. Select Auto-assign IPs, Save, Close.     This allows the bastion host to obtain a public IP address when you create it in this subnet. 
-	On the Route Table tab, choose Edit. For Change to, select the route table ID that you noted earlier and choose Save

### Create Security Groups
You have to create two security groups
-	A security group for EC2 instances 
-	A security group for the Application Load Balancer

### To create two security groups
-	Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/. 
-	In the navigation pane, choose Security Groups, Create Security Group. 
-	For Security group name, enter a name for the security group that's easy to identify and will differentiate it from the other, such as MiranzHyperledgerG1 You use these names later. For Description, enter a brief summary. 
-	For VPC, select the VPC that you created earlier. 
-	Choose Create. 
-	Repeat the steps above to create the other security group.
### Add inbound rules to the security group for EC2 instances
-	Select the security group for EC2 instances that you created earlier
-	On the Inbound tab, choose Edit. 
-	For Type, choose All traffic. For Source, leave Custom selected, and then choose the security group you are currently editing from the   list, for example, MiranzHyperledgerG1. This allows the EC2 instances in the security group to communicate with one another. 
-	Choose Add Rule. 
-	For Type, choose All traffic. For Source, leave Custom selected, and then choose the security group for the Application Load Balancer   from the list, for example, MiranzHyperledgerG2. This allows the EC2 instances in the security group to communicate with the             Application Load Balancer. 
-	Choose Save. 
###Add inbound and edit outbound rules for the security group for the Application Load Balancer
-	Select the security group for Application Load Balancers that you created earlier

-	###On the Inbound tab, choose Edit and then add the following inbound rules: 
-	For Type, choose All traffic. For Source, leave Custom selected, and then choose the security group you are currently editing from the   list, for example, MiranzHyperledgerG2. This allows the Application Load Balancer to communicate with itself and with the bastion       host. 
-	Choose Add Rule. 
-	For Type, choose All traffic. For Source, leave Custom selected, and then choose the security group for EC2 instances from the list,     for example, MiranzHyperledgerG1. This allows the EC2 instances in the security group to communicate with the Application Load           Balancer and the bastion host. 
-	Choose Add Rule. 
-	For Type, choose SSH. For Source, select My IP, which detects your computer's IP CIDR and enters it. 
**_Important_**
   This rule allows the bastion host to accept SSH traffic from your computer, enabling your computer to use the bastion host to view      web interfaces and connect to EC2 instances on the Fabric network. To allow others to connect to the Fabric network, add them as        sources to this rule. Only allow inbound traffic to trusted sources. 
-	Choose Save. 

-	On the Outbound tab, choose Edit and delete the rule that was automatically created to allow outbound traffic to all IP addresses. 
-	Choose Add Rule. 
-	For Type, choose All traffic. For Source, leave Custom selected, and then choose the security group for EC2 instances from the           list(MiranzHyperledgerG1). This allows outbound connections from the Application Load Balancer and the bastion host to EC2 instances     in the fabric network. 
-	Choose Add Rule. 
-	For Type, choose All traffic. For Source, leave Custom selected, and then choose the security group you are currently editing from the   list, for example, MiranzHyperledgerG2. This allows the Application Load Balancer to communicate with itself and with the bastion       host. 
-	Choose Save. 


### Create an IAM Role for Amazon ECS and an EC2 Instance Profile
  When you use this template, you specify an IAM role for Amazon ECS and an EC2 instance profile. The permissions policies attached to     these roles allow the AWS resources and instances in your cluster interact with other AWS resources. For more information, see IAM       Roles in the IAM User Guide. You set up the IAM role for Amazon ECS and the EC2 instance profile using the IAM console                   (https://console.aws.amazon.com/iam/). 
**_To create the IAM role for Amazon ECS_**
-	Open the IAM console at https://console.aws.amazon.com/iam/. 
-	In the navigation pane, choose Roles, Create Role. 
-	Under Select type of trusted entity, choose AWS service. 
-	For Choose the service that will use this role, choose Elastic Container Service. 
-	Under Select your use case, choose Elastic Container Service, Next:Permissions. 
-	for Permissions policy, leave the default policy (AmazonEC2ContainerServiceRole) selected, and choose Next:Review. 
-	For Role name, enter a value that helps you identify the role, such as ECSRoleForMiranzHyperledger. For Role Description, enter a brief summary. Note the role name for later. 
-	Choose Create role. 
-	Select the role that you just created from the list. If your account has many roles, you can search for the role name
-	Copy the Role ARN value and save it so that you can copy it again. You need this ARN when you create the fabric network.

**_The EC2 instance profile that you specify in the template is assumed by EC2 instances in the Fabric network to interact with other AWS services. You create a permissions policy for the role, create the role (which automatically creates an instance profile of the same name), and then attach the permissions policy to the role._**

### To create an EC2 instance profile
-	In the navigation pane, choose Policies, Create policy. 
-	Choose JSON and replace the default policy statement with the following JSON policy:
**_{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:DescribeImages",
                "ecr:BatchGetImage",
                "s3:Get*",
                "s3:List*"
            ],
            "Resource": "*"
        }
    ]
}_**
  
-	Choose Review policy. 
-	For Name, enter a value that helps you identify this permissions policy, for example MiranzHyperledgerPolicyForEC2.. For Description,   enter a brief summary. Choose Create policy. 
-	Choose Roles, Create role. 
-	Choose EC2, Next: Permissions. 
-	In the Search field, enter the name of the permissions policy that you created earlier, for example MiranzHyperledgerPolicyForEC2.. 
-	Select the check mark for the policy that you created earlier, and choose Next: Review.
-	For Role name, enter a value that helps you identify the role, for example EC2RoleForMiranzHyperledger . For Role description, enter a   brief summary.Choose Create role. 
-	Select the role that you just created from the list. If your account has many roles, you can enter the role name in the Search field. 
-	Copy the Instance Profile ARN value and save it so you can copy it again. You need this ARN when you create the fabric network.


### Now to create the stack of fabric template:
**_Note: Use such region to launch instance in which you have created keyPair.
   I have used this region Launch in US East (Ohio) region (us-east-2).
  Just go to this link and fillup the stack form to submit then after 5 min your instance will be created._**





### We have successfully created EC2 instance of fabric based template. The things we have to configure for Development environment are:
-	ssh -i "AQEELKAZMI-key-pair-ohio.pem" ec2-user@ec2-18-218-77-191.us-east-2.compute.amazonaws.com
-	curl -sSL https://goo.gl/6wtTN5 | bash -s 1.1.0
-	mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers
-	curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
-	tar -xvf fabric-dev-servers.tar.gz

-	### sudo  npm install -g composer-cli
If installing composer-cli pass error “Cannot find module './api”
-  cd /usr/lib/node_modules/composer-cli
-	 sudo npm install node-report --unsafe-perm

-	sudo npm install -g composer-rest-server
-	sudo npm install -g generator-hyperledger-composer
-	sudo npm install -g yo
-	composer-playground

### Inside fabric-dev-servers when you run CreatePeerAdminCard.sh , there are chances that some error occurs, to resolve this 
-	cd NodeModules
-	sudo npm rebuild --unsafe-perm



## Hope all errors are removed now and environment is successfully setup.

