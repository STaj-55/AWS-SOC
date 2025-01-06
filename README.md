# AWS-SOC
This project is to simulate a Security Operations Center in an AWS cloud environment.

## Scope of our Project

This lab will go include the following:
- Traffic Monitoring: Inspect and analyze network traffic
- Threat Detection: Set up alerts and rules for malicious activity
- Incident Response: Simulate and respond to security incidents

## Diagram of Project

## Walkthrough

Before we go through to our project, we need to make sure of the following:
- Secure your Root Account with MFA
- Have a dedicated IAM user for administrative tasks
- Enable billing alerts to track costs and avoid overspending

These aren't too hard to set up and are typically the first thing you do after signing up. In my case I have already completed them so we're going to movew on.

Before we move onto setting up our infrastructure, we are going to enable CloudTrail.

### 1. CloudTrail

We are going to setup a CloudTraill Trail to log all API activity across the account. We can do this by accessing CloudTrail from the Console and following through the quick trail create.

![aws1](https://github.com/user-attachments/assets/8ceb2b3d-fe6a-4f56-a417-f2612403d8b3)
When first accessing CloudTrail, we can select "Create a trail"

![aws2](https://github.com/user-attachments/assets/d6a05132-8461-4311-8cbf-12e51a429ed0)
We will be sent to a new page where we can name our trail. I decided to name it as soc-ctlogs. After naming it, we can press create to finalize our trail.

![aws3](https://github.com/user-attachments/assets/4604f153-aa0d-48a5-966d-26517f92dcb0)
As we can see, the trail has been successfully created and has an associated S3 bucket. This is perfect because we can use this to track our logs and usage.

### 2. SOC Lab Budget

![aws4](https://github.com/user-attachments/assets/b34e00f0-4c0f-49e5-9a48-cc272af19b5b)
We can setup a budget through **Resource Group & Tag Editor** in the management console.

![aws5](https://github.com/user-attachments/assets/6c24999b-d56b-4124-b7ec-1fde59edc33c)
Once here, we can assign our tagging policy. I chose to use the following:
- Key: *Project*
- Value: *SOC-Lab*
- Group Name: *Security Operations*

![aws6](https://github.com/user-attachments/assets/9db290fe-46b1-4b85-918e-e977095bb91d)
Now that we have our budget setup, we can effectively tag the services we use for this project to enable easier cost tracking and management.

### 3. Setting up our Infrastructure

We will be building our foundational network and compute resources for this SOC environment in AWS. We will start this off by creating a *Virtual Private Cloud (VPC)*.

#### 3.1 Virtual Private Cloud

We will head to *VPC* in the management console and create a VPC

![aws7](https://github.com/user-attachments/assets/26d90fb7-6c22-4929-a35f-f6b5fbb24f1a)

When creating our VPC we are going to select and input the following:
- Resources to create: VPC Only
- IPv4 CIDR Block: IPv4 CIDR Manual Input
- IPv4 CIDR: 10.0.0.0/16
- Tenancy: Default
Tags Key
- Key: Project
- Value: SOC-Lab

Everything is its own default values, the tags section at the bottom is what were are copying from our Budgeting section.

![aws8](https://github.com/user-attachments/assets/dfad2884-36f4-4065-b0f1-763bee2480d6)

After having your setup configured, press *Create VPC* to create the environment. We will now Create our two subnets, one public and one private.

To create a subnet, look to the left side of the *VPC Console* and press Subnets. From there we can select *Create Subnet*.

![aws9](https://github.com/user-attachments/assets/543c0e3e-e9be-46a7-a2c4-a327010239a7)

For our public subnet, we will configure it with the following:
- Subnet Name: SOC-Pub-Sub-1
- Availability Zone: us-east-1a
- IPv4 VPC CIDR Block: 10.0.0.0/16
- IPv4 Subnet CIDR Block: 10.0.1.0/24

![aws10](https://github.com/user-attachments/assets/da112d47-6681-456f-bab9-666415218a91)

Our private subnet, will follow a similar configuration:
- Subnet Name: SOC-Priv-Sub-1
- Availability Zone: us-east-1a
- IPv4 VPC CIDR Block: 10.0.0.0/16
- IPv4 Subnet CIDR Block: 10.0.2.0/24

Next we are going to create an *Internet Gateway*. Just like before, look towards the left side of the *VPC Console* and select Internet Gateway. From there we can select *Create Internet Gateway*

![aws11](https://github.com/user-attachments/assets/1faae275-ebef-4333-8ec6-88a2e74e1959)

We will assign it the following:
- Name tag: SOC-Internet-Gateway
Tags:
- Key: Name | Value: SOC-Internet-Gateway
- Key: Project | Value: SOC-Lab

After creating it, we can then attach the IGW to our VPC. Just simply select your VPC, and attach. Then once you're back on the IGW page, the status should be green and says Attached.

![aws12](https://github.com/user-attachments/assets/cad80540-17f4-43d8-8634-bf6546f5235b)

Lastly, we are going to create our Route Tables. We will create two route tables, one for each of our subnets.

![aws13](https://github.com/user-attachments/assets/fda6bc02-fec8-49e2-b16e-c35c75f0e22e)

Creating routing tables are quite simple, just like before we will head to the Routing Table section, select create Routing Table, and set up the inital configurations:
- Name: SOC-RT-Pub | SOC-RT-Priv
- VPC: Our VPC

![aws14](https://github.com/user-attachments/assets/a074aa8e-0b04-4a83-b6bc-e08c156f90ad)

After creating our public routing table, we will select the Public Routing Table, Go to Routes, and select Edit Routes.

Here we will add in 0.0.0.0/0 and select our Internet Gateway.

![aws15](https://github.com/user-attachments/assets/7645ebcb-3d97-48dd-81ba-8faf2903f8b7)

The last thing we will do is select your designated routing table (Public or Private), head to Subnet Associations, select edit subnet associations, and then assign it to its corresponding subnet.



