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

![aws2](https://github.com/user-attachments/assets/2137c394-8772-4d39-8539-e6f4f6947527)
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

#### 3.2 EC2 Instances

*I apologize if I may have forgotten something, my power went out so I had to re-write this walkthrough from this point on*

Now that our network is setup, let's configure our *Elastic Compute Cloud* instances to setup our servers for this lab. We will have 3 servers available to us, A public web server, a private app server, and a Bastion Host to log in from.

To create an EC2 instance, we can do so by heading over to EC2 from the management console and selecting *Launch Instance*
![aws16](https://github.com/user-attachments/assets/0cd0d199-62fd-4fe1-8522-6ffd7b842ef0)

Here are the following configurations that I used for our EC2 instances:

*Note for the simplicity of this lab, I only used a single Key Pair, but if you wanted to strengthen the security of your lab, use separate keys*

**Public Web Server**
- Name: SOC-WS
- Amazon Linux
- AMI: Amazon Linux 2023 AMI
- Instance type: t2.micro
- Key Pair Name: SOC-Lab
- Network: SOC-VPC
- Subnet: SOC-Pub-Sub-1
- Auto Assign Public IP: Enabled
Create Security Group
- Security Group Name: SOC-Web-SG
- Description: EC2 Instance for Public Web Server - SOC-Lab
Rules:
- SSH | TCP | Port 22| Source: SOC-Bastion-SG (Most likely will need to add this after)
- HTTP | TCP | Port 80 | Source: Any
- SSH | TCP | Port 22 | Source: My IP

**Private Application Server**
- Name: SOC-AS
- Amazon Linux
- AMI: Amazon Linux 2023 AMI
- Instance type: t2.micro
- Key Pair Name: SOC-Lab
- Network: SOC-VPC
- Subnet: SOC-Priv-Sub-1
- Auto Assign Public IP: Disabled
Create Security Group
- Security Group Name: SOC-App-SG
- Description: EC2 Instance for Private App Server - SOC-Lab
Rules:
- All Traffic | All | All | Source: SOC-Web-SG
- SSH | TCP | 22 | Source: SOC-Web-SG
- SSH | TCP | 22 | Source: SOC-Bastion-SG

**Bastion Host**
- Name: SOC-BH
- Amazon Linux
- AMI: Amazon Linux 2023 AMI
- Instance type: t2.micro
- Key Pair Name: SOC-Lab
- Network: SOC-VPC
- Subnet: SOC-Pub-Sub-1
- Auto Assign Public IP: Enabled
Create Security Group
- Security Group Name: SOC-Bastion-SG
- Description: EC2 Instance for Bastion Host - SOC-Lab
Rules:
- SSH | TCP | Port 22 | Source: My IP

#### 3.3 Verifying our Instances

Now that we have our EC2 Instances up and running, let's SSH into each of them to ensure we can they are operational.

***Before you can use your Key, you must provide the proper permissions, here is the command to use:***
***sudo chmod 400 SOC-Key.pem***

There are different ways to go about this but I will be SSH-ing into them via my Kali Linux VM (Hosted by VMWare Workstation 16 Player)

In order to SSH into an EC2 instance we can use the following command:
- ***ssh -i SOC-Key.pem ec2-user@WebServerPublicIP***

![aws18](https://github.com/user-attachments/assets/8708d28f-d2a3-4fb3-b5bd-5934a2f87728)

As we can see we have successfully logged onto our Web Server from our Kali Linux VM. Now to go from our Web Server to Private App server, we would need to transfer our key.

Here's the command to transfer your SSH Key:
- ***scp -i SOC-Key.pem SOC-Key.pem ec2-user@WebServerPublicIP:~/***

However, since this is our public Web Server, we should be securely storing this key. We can do this by running the following commands:
- ***mkdir -p ~/.ssh***
- ***mv SOC-Key.pem ~/.ssh***
- ***chmod 700 ~/.ssh***

Now that we have secured the location of our key, let's verify it!

![aws19](https://github.com/user-attachments/assets/0509ac78-bd86-43c6-8433-1c9d13fb5163)

![aws20](https://github.com/user-attachments/assets/ccda0a6f-8089-4610-8f07-b9898f3c1943)

As we can see from our screenshots above, the directory itself cannot be access by anyone except the owner, and the file itself is secure.

Now that we have everything SSH wise setup here, let's configure our web server. This can be done with a few commands:
- ***sudo yum install httpd -y***
- ***sudo systemctl start httpd***
- ****echo "SOC Lab Web Server" | sudo tee /var/www/html/index.html***

These commands above install Apache onto our webserver, starts it, and adds the content "SOC Lab Web Server" to the index file which should be displayed on our webpage.

If we use a web browser and head to the public IP of our server, we should be able to see it.

![aws21](https://github.com/user-attachments/assets/717f01b5-9039-4bba-924d-c3bcc7a4d108)

As seen from the screenshot above, our webpage is fully functional and our changes have been applied.

### 4. Enable Monitoring and Logging

In this section we will set up and configure monitoring and logging for our lab. This is essential for a SOC becasuse we need to be able to see what happens in our network.

We will perform the following:
1. **Monitor VPC Network Traffic**, by using VPC Flow Logs to capture network traffic
2. **Track API Calls**, by using CloudTrail to log all API calls made from our account
3. **Threat Detection**, by using Amazon GuardDuty to detect suspicious activity or security threats
4. **Centralized Log Management**, by using Amazon CloudWatch for real-time analysis.
 - I will also be setting up ELK (ElasticSearch, LogStash, and Kibana) to show what a 3rd-party SIEM would look like on AWS 

#### 4.1 VPC Flow Logs

As mentioned before, VPC Flow logs will be used to capture network traffic, which we can then use for network analysis.

In order to set it up, we can head to the VPC Console, once here, select your VPC. Afterwards, head to Actions and left click it to bring the drop-down menu and select Create Flow Log.

![aws22](https://github.com/user-attachments/assets/f1bd42a5-8161-4228-99fa-f695563b8d94)

Let's apply the following configurations:
- Name: SOC-VPC-Flowlog
- Filter: All
- Maximum Aggregation Interval: 1 Minute
- Destination: Send to CloudWatch Logs
- Destination Log Group: /aws/vpc/flowlogs/SOC-Lab
- Service Access: Create and use a new service role
- Service Role Name: SOCFLowLogsRole
- Log Record Format: AWS Default Format
Tags
- Key: Name | Value: SOC-Flow-Logs
- Key: Project | Value: SOC-Lab
- Key: Environment | Value: Development

Now since we created a new service role, let's add in our IAM Policy for this role.

We can do this by heading to IAM > Roles > SOCFLowLogsRole > Add Permissions > Create Inline Policy

Once here switch from Visual to JSON and paste the following policy:

~~~json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"logs:CreateLogStream",
				"logs:PutLogEvents"
			],
			"Resource": "arn:aws:logs:us-east-1:307946648303:log-group:*:log-stream:*"
		},
		{
			"Effect": "Allow",
			"Action": [
				"logs:DescribeLogStreams",
				"logs:CreateLogGroup"
			],
			"Resource": "arn:aws:logs:us-east-1:307946648303:log-group:*"
		},
		{
			"Effect": "Allow",
			"Action": "logs:DescribeLogGroups",
			"Resource": "*"
		},
		{
			"Effect": "Allow",
			"Action": [
				"logs:PutRetentionPolicy",
				"logs:DeleteLogGroup"
			],
			"Resource": "arn:aws:logs:us-east-1:307946648303:log-group:*"
		}
	]
}
~~~

Once this is completed we can press Next and Save Changes.

We can now verify that our logs are being captured in CloudWatch. From the Management Console we can head to CloudWatch, then to Log Groups and select our VPC FLow Log.

![aws23](https://github.com/user-attachments/assets/1c1dd346-4e73-42db-99a4-c3c29b61aa77)

As seen in the screenshot above, logs are properly being captured as shown in the Log Streams tab. 

#### 4.2  CloudTrail

CloudTrail will be used in this lab to log all API activity which is ideal for auditing and security proposes.

Luckily for us, we already set up a CloudTrail in the beginning of our walkthrough so we won't have to spend much time here.

We will be strengthening our CloudTrail by enabling a few settings.

First from the managment console, lets head to CloudTrail, and select the trail we previously made.

Once here we should see general details, and in this tab we will press edit and enable:
- **Log File Validation**

After enabling Log File Validation, the next thing we need to do is set up CloudWatch Logs, this is right underneath General Details and we can press edit to adjust our settings.

From here can do the following:
- Enable CloudWatch Logs
- Log Group: New
- Default Log Group Name
- IAM Role: Create New Role
- Role Name: CT4CW-Role
- Save Changes

After setting this up, we can quickly head back to IAM > Roles > CT4CW-Role and add a new inline policy. Just like before we will switch to JSON and paste the following policy.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCreateLogGroup",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:us-east-1:307946648303:log-group:aws-cloudtrail-logs-307946648303-*"
        },
        {
            "Sid": "AllowCreateLogStream",
            "Effect": "Allow",
            "Action": "logs:CreateLogStream",
            "Resource": "arn:aws:logs:us-east-1:307946648303:log-group:aws-cloudtrail-logs-307946648303-*:log-stream:*"
        },
        {
            "Sid": "AllowPutLogEvents",
            "Effect": "Allow",
            "Action": "logs:PutLogEvents",
            "Resource": "arn:aws:logs:us-east-1:307946648303:log-group:aws-cloudtrail-logs-307946648303-*:log-stream:*"
        },
        {
            "Sid": "AllowDescribeLogGroups",
            "Effect": "Allow",
            "Action": "logs:DescribeLogGroups",
            "Resource": "*"
        }
    ]
}

```

After doing this all we need to do is save it, and add a name for this policy. I named mine: **SOC-CWLogs**.

After setting up CloudWatch Logs we will backout to where we were and scroll down until we see Data Events.

As seen in the screenshot below, we enabed the following configurations:
- Data Events: Enabled
- Resource Type: S3
- Log Selector Template: Log All Events
- Sector Name: SOC-Lab-S3-DataEvents

![aws24](https://github.com/user-attachments/assets/af8dfea3-bada-4d1d-9eac-f31ba941b7b1)

We are now done settingup CloudTrail and can move onto Threat Detection.

#### 4.3 Amazon GuardDuty

GuardDuty is AWS's threat detection service. It's enabled for our lab to detect potential threats within the SOC-Lab environment. Findings are actively monitored for suspicious activities, such as reconnaissance attempts or unauthorized access.

Settings up GuardDuty is extremely easy as it is only a few clicks of a button.

***Note: GuardDuty is a paid service from AWS however we can utilize a free 30-day trial to test it out***

To set up GuardDuty, we can type in GuardDuty from the Management Console and enter the service.

![aws25](https://github.com/user-attachments/assets/92fd517d-a86a-431b-871e-3794e3d7149f)

From here we can press Get Started to start our setup.

![aws26](https://github.com/user-attachments/assets/252aaf36-f52a-49ca-9acb-7d1c3a929d51)

GuardDuty automatically anlayzes data from our VPC FLow Logs, CloudTrail and even DNS logs, so we don't need to worry about setting anything up. All we need to press enable and GuardDuty will be activated.

GuardDuty has a similar layout to most other security solutions and services, displaying vital information such as any threat findings, attack sequences, and as well as a Dashboard that you can set up to see the metrics you want to see.

![aws27](https://github.com/user-attachments/assets/0db9bf02-c7ed-4944-aab3-766c5e18b1ec)

#### 4.4 Centralized Log Management

In this section, we will be setting up a SIEM (Security Information and Event Management) tool to consolidate all of our logs for easier analysis. There are two options that we will be exploring in this lab:
1. Amazon CloudWatch Logs
2. ELK (Elasticsearch, Logstash, Kibana)

#### 4.4.1 Amazon CloudWatch

Amazon CloudWatch is not necessarily a SIEM in its entirety but it does have useful features when combined with other services to provide SIEM like capability.

Luckily for us, we already setup our logs to send to CloudWatch so its as simple as going to Log Groups and selecting which logs we want to view.

To gather more insights about our logs, we can use Log Insights to query for select information about our logs

![aws28](https://github.com/user-attachments/assets/f11c08cf-8b99-4899-838d-7d50a78d01c9)

![aws29](https://github.com/user-attachments/assets/852a8943-e034-4f6b-91de-0d28f580ff5a)

The next thing we are going to do with Amazon CloudWatch is set up an alert. Alerts are great for detecting when a specific incident happens, allowing for faster response.

We'll create an alert for Rejected SSH attempts from our logs. To do this we will set up a metric filter and an alarm.

To create a metric filter, we can head to Log Groups then select our VPC Log. Once here we can select Actions to bring out the drop down menu and we will select *Create Metric Filter*

![aws30](https://github.com/user-attachments/assets/df70f6f9-a8a4-425b-8481-3b41b00f9995)

The first step for creating a metric filter will be Defining and Testing a pattern

The filter pattern we will be using to detect failled SSH attempts is as follows:
- [version, account_id, interface_id, srcaddr, dstaddr, srcport, dstport, protocol, packets, bytes, start, end, action, log_status]

To test this data we will select custom log data and paste this sample entry:
- 2 123456789012 eni-1235b8ca123456789 203.0.113.12 192.168.1.10 54321 22 6 5 500 1623101047 1623101107 REJECT OK

You will be able to test the filter on this page and there should be a single test result shown, as seen in the screenshot below.

After moving onto the next page, you will now be naming the filter and providing some details for it. Here is the configurations I used:
- Filter Name: Failed SSH Filter
- Metric Namespace: VPCFlowLogs/SOC
- Metric Name: Failed SSH
- Metric Value: 1
- Default Value: 0
- Unit: Count

From here we can then move onto the last page where we can review our filter. Once everything is reviewed and looks good, we can create our metric filter.

With our metric filter successfully created, let's create an alarm. This is AWS' version of alerts and we can use these to inform us of whenever an issues arises.

We will now leave out of Log Groups and head into Alarms and create Alarm.

Creating an alarm is quite simple, you will need to select a metric of yours that you would like to create an alarm around. In our case, we will use our Failed SSH Filter.

Here is my configurations:

Metric
- Namespace: VPCFlowLogs/SOC
- Metric Name: FailedSSH
- Statistic: Average
- Period: 1 Minute

Conditions:
- Threshold: Static
- Define Alarm Condition: Greater Than or Equal To
- Define Alarm Value: 1

Notification:
- Alarm State Trigger: In Alarm
- Send an SNS Notification to the following topic: Create a new topic
- Notification Name: SOC-Lab-CW-FailedSSH
- Notification Email: Your email
- Create topic
*Note: There are other options available for you to use, I will be using this for the sake of the lab.*

Name and Description:
- Name: FailedSSH-ALARM
- Description: This alarm will notify you whenever an SSH attempt fails

Save Alarm

Congratulations you now have an alarm setup on AWS! Let's go ahead and test it out!

To do so I will attempt to ssh into the public IP from my laptop (not the same IP of my desktop). This should trigger the alarm.

As seen in the screenshots below the alarm works as expected. When trying to SSH from a machine that is not approved, the alarm was trigged, creating a metric in our logs, as well as sending me an email about the incident.

<img width="1420" alt="Screenshot 2025-01-07 at 4 06 22 PM" src="https://github.com/user-attachments/assets/89dc9f1f-d8d6-4b9c-84a0-eea690989e84" />

<img width="760" alt="Screenshot 2025-01-07 at 4 10 19 PM" src="https://github.com/user-attachments/assets/b36f031e-84a2-4f53-866a-a793b8aa2dc7" />

Amazon GuardDuty is now setup and have successfully performed threat monitoring for this portion of our lab. 

Before we move onto Incident Response and Threat Detection Simulations, let's set up another well known SIEM to show these employers we know what we're talking about. Haha jk 😅... not really 😐 stop playing with my applications.

#### 4.4.2 ELK SIEM

ElasticSearch, Logstash, and Kibana is another industry go-to for a SIEM solution and it has a lot of uniqueness to it that I am fond of. Let's set this thing up in AWS from cloud up!

Let's manually deploy ELK on an EC2 instance.

Here's my configurations:
- Name: ELK
- Amazon Linux 2 AMI
- t3.medium (Although this isn't free tier eligble we can still use this and close the instance when not in use)
- Key Pair: SOC-Key
- Our VPC
- Subnet: SOC-Pub-Sub-1 
- Auto-assign Public IP: Enable 
- Firewall security group: Create new group
- Security Group Name: SOC-Elastic-SG
- Description: Elastic SIEM Security Group
Inbound Security Group Rules
- Type: SSH : Protocol: TCP | Port Range: 22 | Source Type: Custom | Source: App / Web / Bastion Security Group | Description: SSH
- Type: Custom TCP : Protocol: TCP | Port Range: 5044 | Source Type: Custom | Source: App / Web / Bastion Security Group | Description: Logstash
- Type: Custom TCP : Protocol: TCP | Port Range: 9200 | Source Type: Custom | Source: App / Web / Bastion Security Group | Description: Elasticsearch
- Type: Custom TCP : Protocol: TCP | Port Range: 5601 | Source Type: Custom | Source: App / Web / Bastion Security Group | Description: Kibana
Configure Storage
- 1x 20 GiB gp3

After setting our config, we  are ready to launch our instance. With our Elk instance up, we can SSH into it from one of our other instances.

Once we have logged in, we are first going to update our instance, as well as install all of the necessary libraries for ELK.

Here are the commands we will be running:
- sudo yum update -y
- sudo yum install -y java-11-amazon-corretto
- sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
- echo "[elasticserach-7.x]
  name=Elasticsearch repository for 7.x packages
  baseurl=https://artifacts.elastic.co/GPG-KEY-elasticsearch
  enabled=1
  autorefresh=1
  type=rpm-md" | sudo tee /etc/yum.repos.d/elasticsearch.repo
- sudo yum install -y elasticsearch logstash kibana

After installing everything necessary for ELK, we are now going to start our services.
- sudo systemctl start elasticsearch
- sudo systemctl enable elasticsearch
- sudo systemctl start logstash
- sudo systemctl enable logstash
- sudo systemctl start kibana
- sudo systemctl enable kibana

Let's enable these services and check the status of them to ensure they are operational

Now that we have it up, all we need to do now is setup the configuration

Let's first configure Elasticserach, we can edit Elasticsearch configuration with the command:
- sudo nano /etc/elasticsearch/elasticsearch.yml

In the config file, we will update the following settings:
~~~yaml
cluster.name: soc-lab-cluster
node.name: elk-node
network.host: 0.0.0.0
discovery.seed_hosts: []
cluster.initial_master_nodes: ["elk-node"]
~~~


