# Hack-a-Mae 3.0 Environment Setup


## Introduction
This repository contains code and instructions for teams to setup their AWS environment for the Hack-a-Mae 3.0 event.

Each team should receive an email with instructions on how to login to your AWS burner account. Note that access to the AWS burner account is not permitted from the Fannie Mae network, so you should forward the email to your personal email address and use your personal computer for access.

Note that AWS burner accounts are only available for 3 days, so each team will receive 2 accounts. The first account is active April 12-15, and the second account is active April 15-18. Please plan accordingly to move your work to the second account on April 15 and use that account for presentations on April 17.


## Setup Instructions

1. Click on the link provided in the email, review/accept the terms and conditions, and join the event. Links on the left side panel lead to AWS console and credentials if you choose to connect locally.

2. Open the AWS console and change region to us-east-1.

3. Create roles
  - Open the IAM service and select Roles
  - Create admin role for EC2 access
    - Click Create Role
    - Select EC2 use case and click Next ![Screenshot of EC2 use case](/resources/create-ec2-role-1.png)
    - Add Administrator permission and click Next ![Screenshot of EC2 permission](/resources/create-ec2-role-2.png)
    - Enter MyEC2AdminRole for the role name and click Create Role ![Screenshot of EC2 role](/resources/create-ec2-role-3.png)
  - Create admin role for CloudFormation access
    - Click Create Role
    - Select CloudFormation use case and click Next ![Screenshot of CloudFormation use case](/resources/create-cloudformation-role-1.png)
    - Add Administrator permission and click Next ![Screenshot of CloudFormation permission](/resources/create-cloudformation-role-2.png)
    - Enter MyCloudFormationAdminRole for the role name and click Create Role ![Screenshot of CloudFormation role](/resources/create-cloudformation-role-3.png)

4. Create a personal account on [GitHub](https://github.com) if you do not already have one.

5. Go to [HackAMae-3.0-Starter](https://github.com/r2ucrk/HackAMae-3.0-Starter/tree/main) repository and fork to your personal account.

6. Update params.json to set values specific to your AWS burner account (available from VPC service in AWS console):
  - VPC id ![Screenshot of VPC](/resources/vpc.png)
  - Subnet ids for 2 different subnets of your choosing ![Screenshot of subnets](/resources/subnets.png)
  - Default security group id ![Screenshot of security groups](/resources/security-group.png)
  - Team number (1-20)
    

7. Create a new CI/CD pipeline
  - Open the CodePipeline service and click Create Pipeline
  - Name the pipeline whatever you want and click Next ![Screenshot of CodePipeline step 1](/resources/codepipeline-1.png)
  - Select GitHub (Version 2) source provider
  - Click Connect to GitHub and follow the prompts to connect to your personal account
  - Select the repository and branch that you forked in step 5
  - Select No filter trigger type and click Next ![Screenshot of CodePipline source stage](/resources/codepipeline-2.png)
  - Click Skip build stage
  - Select AWS CloudFormation for the deploy provider
    - Select Create or update a stack for Action Mode
    - Name the stack whatever you want
    - Enter source artifact filename cft-starter.yml
    - Enable configuration file and enter source artifact filename params.json
    - Select the MyCloudFormationServiceRole created in step 3
    - Click Next ![Screenshot of CodePipeline deploy stage](/resources/codepipeline-3.png)
  - Review the settings and click Create Pipeline
  - Pipeline should trigger automatically. Monitor the status in CodePipeline and CloudFormation until stack is created successfully.
  ![Screenshot of CodePipeline success](/resources/codepipeline-success.png) ![Screenshot of CloudFormation success](/resources/cloudformation-success.png)


## Environment Details

The cft-starter.yml template creates 2 Application Load Balancers, each with 2 EC2 instances, one in each subnet that you selected. ALB1 is configured with an AutoScaling group, while ALB2 simply has 2 standalone EC2 instances.
![Screenshot of ALB](/resources/alb.png)
![Screenshot of EC2](/resources/ec2.png)

Each EC2 instance is setup with the following items:
  - Amazon Linux 2 AMI for x86_64 created on Feb 23, 2024
  - EBS root filesystem that comes with the AMI
  - EBS data filesystem mounted as /export/appl/data
  - EFS data2 filesystem mounted as /export/appl/data2 (on standalone instances only)
  - 1 or more security groups
  - 1 or more private IP addresses
  - 1 or more tags
  - Additional software
    - Apache web server with PHP
    - Java 17 Corretto
    - jq
    - SpringBoot demo application deployed to /export/appl/data and run as a service on startup

There is also a web page created to summarize the configuration, read data from several text files, and report the status of the SpringBoot application. To access that page, go to http://`DNS name from your ALB`/index.php
![Screenshot of Autoscaling Status](/resources/status-autoscaling.png)
![Screenshot of Standalone Status](/resources/status-standalone.png)


## Challenge Details

The challenge for Hack-a-Mae 3.0 is to:
  - Identify the latest available Amazon Linux 2 AMI
  - Identify EC2 instances in us-east-1 that are running an older version of the AMI
  - Take a snapshot of the status before AMI update
  - Upgrade each EC2 instance to the latest AMI, while preserving data and configuration, including the following:
    - EBS root filesystem, with any files and packages created outside of the AMI
    - EBS data filesystem
    - EFS data2 filesystem, if present
    - Security group(s)
    - Private IP address(es)
    - Tag(s)
  - Take a snapshot of the status after AMI update
  - Validate the status page for each ALB to confirm the environment is working properly. The only values that should change are Instance Id and AMI Id/description.
  - Build the solution into a automated pipeline capable of running within the Fannie Mae environment.
