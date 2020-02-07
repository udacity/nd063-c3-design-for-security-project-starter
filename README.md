# Cloud Security - Protecting resources and data in the cloud
 
In this project, you will:
 
* Deploy and assess a simple web application environment’s security posture
* Test the security of the environment by simulating attack scenarios and exploiting cloud configuration vulnerabilities
* Implement monitoring to identify insecure configurations and malicious activity 
* Apply methods learned in the course to harden and secure the environment
* Design a DevSecOps pipeline
 

 
## Dependancies and Prerequisites
 
### Access to AWS account  
Students will need to use their personal AWS accounts.  Udacity will provide a $100 credit for any usage costs. If project instructions are followed we do not anticipate usage costs to exceed this amount.
 
### Installation of the AWS CLI and Local Setup of AWS API keys
Instructions and examples in this project will make use of the AWS CLI in order to automate and reduce time and complexity.
Refer to the below links to get the AWS CLI installed and configured in your local environment.
 
[Installing the CL](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html)
 
[Configuring the CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
 
### Local setup of git and GitHub Repository
You will need to clone / fork / download [this GitHub repo](https://github.com/udacity/nd063-c3-design-for-security-project-starter) in order to work on and submit this project.
 
## Exercise 1 - Deploy Project Environment
 
**_Deliverables for Exercise 1:_**
* There are no deliverables for Exercise 1.
 
### Task 1:  Review Architecture Diagram
In this task, the objective is to familiarize yourself with the starting architecture diagram. An architecture diagram has been provided which reflects the resources that will be deployed in your AWS account.
 
The diagram file, title `AWS-WebServiceDiagram-v1-insecure.png`, can be found in the _starter_ directory in this repo.
 
![base environment](/starter/AWS-WebServiceDiagram-v1-insecure.png)
 
#### Expected user flow:
- Clients will invoke a public-facing web service to pull free recipes.  
- The web service is hosted by an HTTP load balancer listening on port 80.
- The web service is forwarding requests to the web application instance which listens on port 5000.
- The web application instance will, in turn, use the public-facing AWS API to pull recipe files from the S3 bucket hosting free recipes. An IAM role and policy will provide the web app instance permissions required to access objects in the S3 bucket.
- Another S3 bucket is used as a vault to store secret recipes; there are privileged users who would need access to this bucket. The web application server does not need access to this bucket.
 
#### Attack flow:
- Scripts simulating an attack will be run from a separate instance which is in an un-trusted subnet.
- The scripts will attempt to break into the web application instance using the public IP and attempt to access data in the secret recipe S3 bucket.
 
### Task 2: Review CloudFormation Template
In this task, the objective is to familiarize you with the starter code get you up and running quickly. Spend a few minutes going through the .yml files in the _starter_ folder to get a feel for how parts of the code will map to the components in the architecture diagram. 
Additionally, we have provided a CloudFormation template which will deploy the following resources in AWS:
 
#### VPC Stack for the underlying network:
* A VPC with 2 public subnets, one private subnet, and internet gateways etc for internet access.
 
#### S3 bucket stack:
* 2 S3 buckets that will contain data objects for the application.
 
#### Application stack:
* An EC2 instance that will act as an external attacker from which we will test the ability of our environment to handle threats
* An EC2 instance that will be running a simple web service.
* Application LoadBalancer
* Security groups
* IAM role
 
## Task 2: Deployment of Initial Infrastructure
In this task, the objective is to deploy the cloudformation stacks that will create the below environment.
 
![base environment](/starter/AWS-WebServiceDiagram-v1-insecure.png)
 
 
We will utilize the AWS CLI in this guide, however you are welcome to use the AWS console to deploy the CloudFormation templates.
 
 
1. From the root directory of the repository - execute the below command to deploy the templates.
 
Deploy the S3 buckets
```
aws cloudformation create-stack --region us-east-1 --stack-name c3-s3 --template-body file://starter/c3-s3.yml
```
 
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-s3/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```
Deploy the VPC and Subnets
```
aws cloudformation create-stack --region us-east-1 --stack-name c3-vpc --template-body file://starter/c3-vpc.yml
```
 
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-vpc/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```
 
Deploy the application stack - you will need to specify a pre-existing key pair name
```
aws cloudformation create-stack --region us-east-1 --stack-name c3-app --template-body file://starter/c3-app.yml --parameters ParameterKey=KeyPair,ParameterValue=<add your key pair name here> --capabilities CAPABILITY_IAM
```
 
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-app/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```
 
Expected example AWS Console status: 
https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks
 
![Expected AWS Console Status](cloudformation_status.png)
 
2. Once you see Status is CREATE_COMPLETE for all 3 stacks, Obtain the required parameters needed for the project.
 
Obtain the name of the S3 bucket by navigating to the Outputs section of the stack:
 
![Outputs Section](s3stack_outputs.png)
 
Note down the names of the two other buckets that have been created, one for free recipes and one for secret recipes.  You will need the bucket names to upload example recipe data to the buckets and to run the attack scripts.
 
You will need the Application Load Balancer endpoint to test the web service - ApplicationURL
You will need the web application EC2 instance public IP address to simulate the attack - ApplicationInstanceIP
You will need the public IP address of the attack instance from which to run the attack scripts - AttackInstanceIP
 
You can get these from the Outputs section of the **c3-app** stack.
 
![Outputs](outputs.png)
 
3.  Upload data to S3 buckets
Upload the free recipes to the free recipe S3 bucket from step 2 that has been by typing this command into the console (you will replace <BucketNameRecipesFree> with your bucket name):
 
Example:  
```
aws s3 cp free_recipe.txt s3://<BucketNameRecipesFree>/ --region us-east-1
 
```
 
Upload the secret recipes to the free recipe S3 bucket from step 2 that has been by typing this command into the console (you will replace <BucketNameRecipesFree> with your bucket name):
 
Example:  
```
aws s3 cp secret_recipe.txt s3://<BucketNameRecipesSecret>/ --region us-east-1
 
```
 
 
4. Test the application
Invoke the web service using the application load balancer URL
http://<ApplicationURL>/free_recipe
You should receive a recipe for banana bread.
 
 
## Task 5:  Identify bad practices
 
Based on the architecture diagram, and the steps you have taken so far to upload data and access the application web service, identify at least 2 obvious poor practices as it relates to security.  Include justification.
 
Deliverable: txt file e1t5.txt
 
# Exercise 2 : enable security monitoring
 
### Task 1: Enable security monitoring using AWS native tools
 
First we will setup security monitoring to ensure that the AWS account and environment configuration is in compliance with the CIS standards for cloud security.
 
Enable AWS Config (skip this step if you already have it enabled)
See below screenshot for the initial settings (config_enable.png)
On the Rules page click Skip
On the Review page click confirm
Enable AWS Security Hub
From the Security Hub landing page click Go To Security Hub
On the next page click Enable Security Hub
Enable AWS Inspector scan
From the Inspector service landing page leave the defaults and click Advanced
![Inspector1](Inspector_setup_runonce.png)
Uncheck All Instances and Install Agents
Choose Name for Key and ‘Web Services Instance - C3’ for value, Hit Next
![Inspector2](Inspector_setup2.png)
Edit the rules packages as seen in the screen shot below
![Inspector3](Inspector_setup3.png)
Uncheck Assessment Schedule
Set a duration of 15 minutes
Enable AWS Guard Duty
 
After 1-2 hours data will populate in these tools giving you a glimpse of security vulnerabilities in your environment.
 
### Task 2: Identify and triage vulnerabilities
 
Please submit screen shots of:
AWS Config - showing non-compliant rules
AWS Inspector - showing scan results
AWS Security Hub - showing compliance standards for CIS foundations.
 
Name the files e2t2_config.png, e2t2_inspector.png, e2t2_securityhub.png
 
Research and analyze which of the vulnerabilities appear to be related to the code that was deployed for the environment in this project.
 
Bonus - provide recommendations on how to remediate the vulnerabilities.
 
Submit your findings in e2t2.txt
 
# Exercise 3 - Attack simulation
 
Now you will run scripts that will simulate the following attack conditions:
Making an SSH connection to the application server using brute force password cracking
Capturing secret recipe files from the s3 bucket using stolen API keys
 
## Task 1: Brute force attack to exploit ssh ports facing the internet and an insecure configuration on the server
 
Login to the attack simulation server using your ssh key pair.
Run this command to start a brute force attack against the application server.  You will need the application serveri hostname for this.
```
hydra -l ubuntu -P rockyou.txt ssh://ec2-52-203-199-229.compute-1.amazonaws.com
```
 
You should see output similar to the following.
		
![Brute Force](brute_force.png)
 
Wait 10 - 15 minutes and check AWS Guard Duty
 
Answer the following questions:
What findings were detected related to the brute force attack
Take a screen shot of the guard duty finding specific to the attack.
Research the AWS Guard Duty documentation page and explain how GuardDuty may have detected this attack - i.e. what was its source of information.
Submit to e3t1_guardduty.png and e3t1.txt
 
## Task 2: Accessing secret recipe data file from S3
 
Imagine a scenario where API keys used by the application server to read data from S3 were discovered and stolen by the brute force attack.  This provides the attack instance the same API privileges as the application instance.  We can test this scenario by attempting to use the API to read data from the secrets S3 bucket.
 
Run the following API call to view and download files from the secret recipes S3 bucket.  You will need the name of the S3 bucket for this.
 
```
# view the files in the secret recipes bucket
aws s3 ls  s3://<BucketNameRecipesSecret>/ --region us-east-1
 
# download the files
aws s3 cp s3://<BucketNameRecipesSecret>/secret_recipe.txt  .  --region us-east-1

# view contents of the file
cat secret_recipe.txt

```
Take a screenshot showing the breach:
E3T2_s3breach.png

Stand Out exercise:
Choose one of the application vulnerability attacks outlined in the OWASP top 10 (e.g. SQL injection, cross site scripting)
Attempt to invoke the application using the ALB URL with a corrupt or malicious URL payload.
Setup the AWS WAF in front of the ALB URL.
Repeat the malicious URL attempts
Observe the WAF blocking these requests.
Submit screen shots of your attempts and monitoring or logs from the WAF showing the blocked attempts.

# Exercise 4 - Implement Security Hardening

## Task 1 - Remediation plan

As a cloud architect you have been asked to apply security best practices to the environment so that it can withstand attacks and be more secure.
Identify 2-3 changes that can be made to our environment to prevent an ssh brute force attack from the internet.
Neither instance should have had access to the secret recipes bucket, in the even that instance API credentials were compromised how could we have prevented access to sensitive data.

Submit to e4t1.txt


## Task 2 - Hardening

### remove ssh vulnerability on the application instance

To disable SSH password login on the application server instance.
```
# open the file /etc/ssh/sshd_config
sudo vi /etc/ssh/sshd_config
# Find this line:
PasswordAuthentication yes
# change it to
PasswordAuthentication no
# save and exit
#restart ssh server
sudo service ssh restart
```

Test that this made a difference.  Run the brute force attack again from Ex. 3 task 1.  

 Take a screenshot of the terminal window where you ran the attack highlighting the remediation (E4T2_ssh.png)

### Apply network controls to restrict application server traffic

Update the security group which is assigned to the web application instance.  The requirement is that we only allow connections to port 5000 from the public subnet where the application load balancer resides.
Test that the change worked by attempting to make an ssh connection to the web application instance using its public URL.
Submit a screenshot of the security group change and your ssh attempt.

### Least privilege access to S3.  

Update the IAM policy for the instance profile role used by the web application instance to only allow read access to the free recipes S3 bucket.
Test the change by using the attack instance to attempt to copy the secret recipes (Ex 3 Task 2).
Submit a screen shot of the new S3 bucket policy and the attempt to copy the files.

### Apply default server side encryption to the S3 bucket

This will cause the S3 service to encrypt any objects that are stored going forward by default.
Use the below guide to enable this on both S3 buckets.   
https://docs.aws.amazon.com/AmazonS3/latest/dev/bucket-encryption.html

Capture the screen shot of the secret recipes bucket showing that default encryption has been enabled.

E4T2_s3steal.png
E4T2_s3policy.png (or c3-app_solution.yml)
E4T2_s3encryption.png (or c3-s3_solution.yml)
E4T2_networksg.png (or c3-app_solution.yml)


### Task 3: Check monitoring tools to see if the changes that were made have reduced the number of findings.

Go to AWS inspector and run the inspector scan that was run in Ex 2.
After 20-30 mins - check Security Hub to see if the finding count reduced.
Check AWS Config rules to see if any of the rules are now in compliance.
Submit screen shots of Inspector and Security Hub and AWS Config



E4T3_securityhub.png
E4T3_config.png
E4T3_inspector.png

### Task 4: Questions and Analysis

What additional architectural change can be made to reduce the internet facing attack surface of the web application instance.
Assuming the IAM permissions for the S3 bucket are still insecure, would creating VPC private endpoints for S3 prevent the unauthorized access to the secrets bucket.
Will applying default encryption setting to the s3 buckets encrypt the data that already exists?
What would happen if the original cloud formation templates are applied to this environment.

E4T4.txt

### Stand Out exercise

Make changes to the environment by updating the cloud formation template.
Brainstorm and list additional hardening suggestions aside from those implemented that would protect the data in this environment.

# Exercise 5 - Designing a DevSecOps pipeline

Take a look at a very simple yet deployment pipeline diagrammed below:

![DevOpsPipeline](DevOpsPipeline.png)

The high level steps are as following:

The user makes a change to the application code or OS configuration for a service.
Once the change is committed to source a build a kicked off resulting in an AMI or a container image.
The infrastructure as code is updated with the new AMI or container image to use.
Changes to cloud configuration or infrastructure as code may have also been committed.
A deployment to the environment ensues applying the changes



## Task 1:  Design a DevSecOps pipeline

Update the starter DevOpsPipeline.ppt (or create your own diagram using a different tool)
At minimum you will include steps for:
Infrastructure as code compliance scanning.
AMI or container image scanning.
Post deployment compliance scanning

Submit your design as a ppt or png image named DevSecOpsPipeline.ppt

## Task 2 - Tools and documentation
      
You will need to determine appropriate tools to incorporate into the pipeline to ensure that security vulnerabilities are found.

Identify tools that will allow you to do the following:
Scan infrastructure as code templates
Scan AMI’s or containers for OS vulnerabilities
Scan an AWS environment for cloud configuration vulnerabilities
For each tool - identify an example compliance violation or vulnerability which it might expose.

Submit your answers in E5T2.txt

## Standout Task

Run an infrastructure as code scanning tool on the cloudformation templates provided in the starter.
Show that the tool has correctly identified bad practices
If you had completed the remediations by updating the cloud formation templates, run the scanner and compare outputs showing that insecure configurations were fixed.

E5T3.png
E5T3.txt

# Clean up 

Once your project has been submitted and reviewed - to prevent undesired charges don’t forget to: 
Disable Security Hub and Guard Duty
Delete recipe files uploaded to the S3 buckets
Delete your cloud formation stacks
