
# CHALO new vertical by Yashodhan Khadilkar

In the following sections, I will explain the understanding of the given scenario, assumptions
and explanation of the architecture diagram, Best practices, and further steps. 

In the cloudfromation folder, please find the CTF templates to deploy the environment. 


## Assumptions

- As per the scenario, we have created a new AWS account. 
    
    - I am assuming that we are starting with a brand new AWS setup. 
    - We should use three different accounts for 3 environments. DEV, STAGE and PROD.
    - To centrally create three different accounts we should utilize the AWS Organizations service.
    - With the help of Organizations you can centrally manage CloudTrail, IAM roles, Users, and Security. 

- Single Account Setup
- Considering deployments in the Dev environment.
- Account Region as 'ap-south-1'
- Service deployment on EC2 instances.
- Using RDS service for Database. 
- Pre-configured Route 53. 
- Creation of High Availability using multi-AZ
- Considering EFS is pre-configured. Only adding root volume to the ec2 instances. 
- I like to create separate cloud formation templates for each resource, to reduce dependency.
- Because of time limitations, I was not able to create an interconnection between cloud formation templates, therefore parameter values might be hardcoded. 
- I have deployed all CFTs in my personal account and validated them. Please find ScreenShot in the root folder. 

    
## Architecture Diagram

Please find the Architecture Diagram in the 'arch_diagram' folder.
In this section, I will majorly describe the Architecture Diagram. 

- 3 Tier Architecture: 
    
    - 1st Tier: Public Subnet

        - Public subnet is connected to the ALB. ALB will receive filtered requests from WAF. 
        - In the public subnet our Nginx proxy servers will reside. 
        - Nginx servers are part of the Auto Scaling Group. Based on the traffic and ec2 utilizations we can set auto-scaling policies. 
        - Nginx servers are part of the Multi-Az deployment for High Availability. 
        - Along with Nginx NatGateway and Bastion host are present in the Public subnet. 
        - NatGateway provides private subnet access to the internet. 
        - Bastion host provides users access to the resources in the private subnet.
        - NatGateway and Bastion hosts are not part of Multi-AZ deployments. 

    - 2nd Tier: Private Subnet1

        - In Private Subnet1, application servers are present. 
        - All tomcat servers are behind the ALB.
        - This internal facing ALB will receive a request from ec2 instances in the public subnet and forward them to the TomCat servers. 
        - TomCat servers are also part of Auto Scaling Group. Based on utilization the number of tomcat servers will increase across multi-az
        - To provide HA tomcat servers are deployed in multi-az
    
    - 3rd Tier: Private Subnet2 

        - In private subnet2, Database servers are present. 
        - All tomcat servers will point to the master RDS instance which is present in az1. 
        - In this architecture, the RDS instance is getting synchronously replicated to another az. 
        - To provide HA RDS instances deployed in multi-az. Which is managed by AWS. 

    
## Best Practices

Please find the list of best practices to follow. 

- WAF and DDoS protection should be enabled to protect the environment from attacks.
- Should only allow HTTPS traffic on the application load balancer. Or redirects from HTTP to HTTPS.
- S3 buckets should be encrypted and should have bucket policies to block and allow only secure traffic. 
- All AWS resources should be encrypted with KMS. 
- It is better to use customer-managed KMS than default KMS. 
- S3 bucket should have logging enabled. 
- S3 buckets should have object versioning and retention in place.
- ALB should have logging enabled to check the requests. 
- RDS should be deployed in the multi-az environment. 
- RDS database logging should be enabled. 
- EBS and EFS volume should be encrypted. 
- RDS instances should have automatic backups
- EC2 instances must have a systems manager enabled. 
- EC2 instance patching activities should be done in an automated and at a weekly cadence with the help of the systems manager. 
- IAM roles should not have an inline policy
- Should use VPC endpoints for S3 bucker operations
- If creating an IAM user- ensure that password expiry is set
- Use of AWS backup server - to automatically backup S3, ebs, efs, ec2 and rds services. 
- Use of resource tagging. 
- utilization of the security hub to keep the track of best practices and automatic resolution. 
- Going forward, use the Audit service to audit the environment. 
- All EC2 instances should use metadata service version 2
- EC2 should be configured to use VPC endpoints


## Further steps

- Since, all the CFTs are standalone. We should make changes and make them dependable. 
- Implementation of the Common platform services mentioned in the Architecture diagram. 
- For logging and monitoring of the Infrastructure implementation of the New Relic agent
- Development of Notification pipeline and Alarm pipeline. 
- CICD pipeline to deploy changes continuously and also in higher environments. 

## Things to Note
- In all CFTs references of resources are given as per my personal account. 
- CFTs do not have output sections. That's why deploying CFT through the console is preferred. 
- Default values that are given in the parameters might not work in your environment. 

## Steps to deploy the Infrastructure

The CFTs which I have developed are majorly one resource per CFT. By dividing one large CFT into multiple CFTs, we can put a termination lock on resources and also on CFTs. 

We have to deploy these CFTs in some specific order: 
    
    1. VPC-Private-Public.yaml
    2. vpc-nat-gateway.yaml
    3. KMS-key.yaml
    4. ec2-iam-role
    5. elb-asg-nginx-tomcat.yaml
    6. rds-postgres.yaml

It is preferred to deploy these CFTs via the AWS console because the output section is not present in CFTs.  But please find below mentioned CLI command to deploy the resources. 

    aws cloudformation deploy --template-file ./CloufFormation/VPC-Private-Public.yaml --stack-name VPCsetup 
