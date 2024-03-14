# Managing Cloud Nodes (Docker & EC2) In Jenkins


## Docker as Cloud Node
Docker Plugin ➜ required

To configure a Docker container as a cloud node in Jenkins, we require the host URI and port number for communication between the host and Jenkins. However, since Jenkins is running within a Docker container itself, we need to introduce a third party to facilitate the connection, which involves creating another Docker container using 'socat.' If this process seems unclear, I recommend the following post for further reading. Nevertheless, proceed with the setup as it may become clearer once implemented.

https://stackoverflow.com/questions/47709208/how-to-find-docker-host-uri-to-be-used-in-jenkins-docker-plugin

Execute the following command on the host machine where your Docker environment is running and where the Jenkins Docker container is located:

```
[orange@centos8 centos7_all]$ docker run -d --restart=always -p 127.0.0.1:2375:2375 --network net4jenkins --name docker4jen -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
```

Verify that the IP address of 'docker4jen' and the port '2375' are correct before proceeding.


Let's follow the following path;

Dashboard ➜  Manage Jenkins ➜ Clouds ➜ New cloud

Complete the fields below.


Cloud name : node1_docker_cloud.

Docker:  if you don’t see this option you need to install plugin.

Click on ‘Docker Cloud Detail’.

Docker Host URI:  tcp://172.18.0.6:2375  

Server credentials :  No Need here

Enabled? :    _[Very Important to tick this]_


click on ‘Test Connection’

Leave everyting default


click Save.




<img src="/part-3/pic-2/jp2-23.png" width="500" />

You will be directed to a completely different page. However, the process is not complete yet; we still need to configure the agent template.

Follow these steps again:

Dashboard ➜ Manage Jenkins ➜ Clouds

Once on the 'Clouds' page, click on the name in my setup 'node1_docker_cloud' , then select 'configure' to return to the page where you can view and adjust the available options.

“Docker Cloud details”

“Docker agent template”

The first option has been configured; now let's proceed to click on the option "Docker Agent templates".




I will outline the settings I have adjusted, while leaving the rest at their default values. I will recommended to follow a similar pattern when configuring your setup.



Labels?  :  agent1_node1_docker_cloud.

Enabled? :   Tick this.

Name?  : template for agent1_node1_docker_cloud.

Docker Image?   : jenkins/agent:latest-alpine-jdk21.

Instance Capacity : 1

Remote File System Root : /home/jenkins

The rest are left as default.



We have completed the setup, but there is a possibility that the job section [when we modify/create a job] may not detect this yet. Therefore, ensure that you have followed all steps correctly, keep both sections enabled, as in  “Docker Cloud details” & “Docker agent template”. 

Copy the label instead of trying to remember it’s name or it’s label name and type in. Now paste it in “Label Expression ” in the job configuration by ticking “Restrict where this project can be run? ” , and wait for it to start the Docker. You should observe the expected outcome.

Click on 'Save'. Once on the job's page, click 'Build Now'. Your 'Build History' may display something similar to the following, but since it needs to initiate a container, it may take approximately a minute. This is why I recommended simply copying the label and pasting it. Initially, it may not appear, and secondly, it takes some time, which could lead one to believe an error was made.

```
(pending—template for agent1_node1_docker_cloud-0002eh9arxhhx is offline) 
```

here is the outcome for my job;

```
Console Output
Started by user admin user
Running as SYSTEM
Building remotely on template for agent1_node1_docker_cloud-0002eh9arxhhx on node1_docker_cloud (agent1_node1_docker_cloud) in workspace /home/jenkins/workspace/test_job
[test_job] $ /bin/sh -xe /tmp/jenkins359106787808240172.sh
+ echo Hello
Hello
+ cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.19.1
PRETTY_NAME="Alpine Linux v3.19"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
Finished: SUCCESS
The shell code that was used in this job;
```
If you wish to access the job's code, it is provided here.

```
echo "Hello"
cat /etc/os-release
```


<img src="/part-3/pic-2/jp2-15.png" width="500" />





## EC2 as Cloud Node:
EC2 Plugin ➜ required

Remain within the same AWS region as you proceed with this exercise.


As you may have inferred, the typical pattern involves configuring the AWS side before proceeding to work within the Jenkins dashboard. To avoid complications, it is advisable to create an EC2 key pair within the same region. While you can generate keys for multiple regions and import them, it is more efficient to stick to one region for all steps in this section. When creating a user or Access Key credentials, you may be directed to a global site regardless. By following this approach, you can prevent unexpected issues and ensure a smooth setup process for your EC2 cloud node.

https://repost.aws/knowledge-center/ec2-ssh-key-pair-regions

#### Amazon EC2 key pair:
Below are the instructions for generating an Amazon EC2 key pair, which will be required later in the Jenkins dashboard.

- Open the Amazon EC2 console  
- In the navigation pane, under Network & Security, choose Key Pairs. 
- Click Create key pair. 
- Provide a descriptive name for the key pair. Amazon EC2 associates the public key with this name. 
- Choose either RSA  as the key pair type. I was using CentOS so I picked pem. 
- Select the format in which to save the private key: 
- To save it in a format compatible with OpenSSH, choose pem. 
- To save it in a format for PuTTY, choose ppk. 
- Optionally, add tags to the public key. 
- Click Create key pair. 

Your browser will automatically download the private key file. Save it securely. 


#### Access Key ID and Secret Access Key:

We will proceed to generate an access key for an existing user for whom I needed to create access keys.

Sign in to the AWS Management Console. 
- Navigate to IAM (Identity and Access Management). 
- Click on “Users” in the left navigation pane. 
- Choose the IAM user for whom you want to create access keys. 
- Go to the “Security credentials” tab. 
- Click “Create access key”. 
- Your new access key pair will be generated. Click “Show Access Key” to view your Access Key ID and Secret Access Key. 

Ensure to securely store your secret access key. These keys are crucial for programmatic access to AWS services and will be necessary when configuring the EC2 cloud in Jenkins.




To begin, launch a Linux-based AMI; for example, I opted for Amazon Linux 2 of type t2.micro to keep costs minimal or potentially free. To create an AMI of an EC2 instance with Java installed, you can either manually start the EC2 instance, log in, install Java, and create an image, or include commands in the 'user data' section during instance creation to install Java. The method chosen does not impact the outcome.


#### Create image:

This exercise consists of two parts. Initially, we must create an ec2 instance if one does not already exist, install the necessary applications on it, and then generate an image from it. When creating an AMI, it's important to note that security groups and access keys are not inherently included in the AMI. However, we will opt for the ones we have recently generated to maintain consistency in the instructions.

Let’s launch an Amazon EC2 t2.micro instance with Amazon Linux 2, configure it with the “temp-ssh” security group, and keep the instructions concise:

Sign in to the AWS Management Console. 

- Navigate to the EC2 dashboard. 
- Click “Launch Instance”. 
- Choose “Amazon Linux 2” as the AMI. 
- Select the t2.micro instance type (eligible for the free tier). 
- In the next step, click “Create security group”  or pick if you already have one.

it must have ssh port 22 open or ICMP if you needed to ping it: 

Choose an existing key pair or create a new one. 

- Configure storage

If you plan to use 'user data,' access 'Advanced Details' by scrolling down to find this option. It's important to be aware that Jenkins may encounter issues with older Java versions, depending on the versions of Jenkins, EC2, or other Docker OSes like CentOS 7/8. If you opt for Amazon Linux 2 or any Red Hat variant, the provided user data should work effectively. However, if you are using a Debian-based or different flavor, you may need to adjust this code accordingly.

```
  # install java;
sudo  yum -y install wget && \
    cd ~ && \
    wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.rpm && \
    sudo yum -y install ./jdk-17_linux-x64_bin.rpm
```

- Click “Launch Instances”. You can Review settings and click “Launch”. 
       
#### Create AMI;

Once in  AWS Management Console. 

- Navigate to the EC2 Dashboard and select “Instances” from the left sidebar. 
- Choose the EC2 instance we created in privous step.  
- Click on “Actions”, then navigate to “Image and templates”, and select “Create image”. 
- In the Create Image dialog box:
- Type a unique name and description for your AMI if you want to. 
- Choose “Create Image”. 
- If you don’t want your instance to be shut down, choose “No reboot”. However, it is not recommended. 
- It takes some time for the image to be create. After a few minutes, your AMI will be created. You can find it in the AMIs view in AWS Explorer next time you create - an instacne . To see your AMIs, choose “Owned By Me” from the Viewing drop-down list. Refresh if needed.
- Remember to terminate any instances you no longer require to avoid incurring unnecessary costs.


Let's return to the Jenkins console/dashboard.
There are three steps to follow: 
- Create credentials for the Access Key.
- Generate a key for the private key (key pair).
- Configure the cloud [Jenkins] settings.

#### Step1:   AWS Credentials for “Amazon EC2 Credentials”

When setting up the EC2 cloud node in Jenkins, you will be prompted to enter your 'Amazon EC2 Credentials.' Therefore, it is essential to complete this step beforehand.

To do so, navigate through the following path:
Dashboard ➜ Manage Jenkins ➜ Credentials ➜ System ➜ Global

Click ‘Add Credentials’

I will outline the settings I have adjusted, while leaving the rest at their default values. I will recommended to follow a similar pattern when configuring your setup.

- Kind: AWS Credentials
- Scope :  Global
- ID :  aws_ec2_ireland
- Description :  aws cred for ireland region
- You hade already create Access Key and Secret Access in Step. Where you were give the only one time option of seeing it an noting it down.
- Access Key ID?:  Type or paste you ID here AKXODLWLXXXXX [enter your own]
- Secret Access Key:  Type or paste you Access Key
- IAM Role Support : leav it Defaults


<img src="/part-3/pic-2/jp2-5.png" width="500" />


<img src="/part-3/pic-2/jp2-7.png" width="500" />

#### step2: Global Credential for “EC2 Key Pair's Private Key”

When creating an EC2 cloud node, you will need to provide the "EC2 Key Pair's Private Key." It is important to set this up beforehand like we did for ec2 credentials.

Navigate through the following path:
Dashboard → Manage Jenkins → Credentials → System → Global

Click ‘Add Credentials’

I will outline the settings I have adjusted for this part too, while leaving the rest at their default values. I will recommended to follow a similar pattern when configuring your setup.

- Kind: SSH Username with private key
- Scope :  Global
- ID :  private_key4ec2_ireland
- Description 
- Username 
- Private Key ➜   Enter directly  ➜   Add
		Paste the contents of the downloaded ec2 key pair into the designated field 




<img src="/part-3/pic-2/jp2-6.png" width="500" />

#### Configure the cloud settings

Dashboard ➜ Manage Jenkins ➜  Clouds ➜ myec2 ➜ Configure

Dashboard ➜ Manage Jenkins ➜ You can proceed to either Node or Cloud from this point. For the initial setup, let's select Cloud, although the choice should not have a significant impact at this stage . ➜ ‘New Cloud’ [This interface may seem familiar as we have previously encountered it in our Docker setup under the Node section.] ➜ Cloud Name: type any unique name ➜ and pick → Amazon EC2 .

I will outline the settings I have adjusted, while leaving the rest at their default values. I will recommended to follow a similar pattern when configuring your setup.

- Name:  myec2
- Amazon EC2 Credentials:  aws_ec2_ireland  
- Region :  eu-west-1
- EC2 Key Pair's Private Key :  private_key4ec2_ireland  

AMIs : 
- Description :  ami custome made
- AMI ID :  ami-086d7535f3b20c7a4
- Instance Type :  t2micro
- Security group names :  temp-sg-ssh-http
- Remote FS root :  /home/ec2-user
- Remote user :  ec2-user
- AMI Type :  unix [as I have picked Amazon Linux]
- Remote ssh port :  22 

Advance:
- Number of Executors :  1
- Java Path :  java [it was default anyway]

Below are critical settings to modify;

- Associate Public IP :  Tick
- Connection Strategy :  Public IP  _[Please make sure it is Public because the def is Private]_
- Host Key Verification Strategy : ‘check-new-hard’


<img src="/part-3/pic-2/jp2-8.png" width="500" />

Patience is key with this option. Consider my stats: the process started at 3:23:29 PM and completed at 3:29:13 PM. If you prefer a quicker option you may pick 'accept new', it is advised to avoid selecting 'accept new' as this may lead to undesired outcomes in terms of security.

- Disk Space Monitoring Thresholds :  Tick
- Free Temp Space Threshold :  0 _[Please note that blank does not mean zero here]_
- Free Temp Space Warning Threshold :  0


<img src="/part-3/pic-2/jp2-13.png" width="500" />

For your initial attempt, I recommend keeping this setting as is. If you encounter an error similar to one in picture 


<img src="/part-3/pic-2/jp2-27.png" width="500" />


Then please refer to the instructions below for the necessary adjustments.

- Save


Below is a snippet outlining the timeline of Jenkins initiating the creation of an EC2 node, establishing a connection, and successfully completing the process. The process began at 3:23:29 PM and concluded by 3:29:13 PM. This exemplifies the importance of patience when working with this setup.


Start Time:

```
Mar 12, 2024 3:23:29 PM hudson.plugins.ec2.EC2Cloud
INFO: Launching instance: i-0ed04daa37674e7f8
Mar 12, 2024 3:23:29 PM hudson.plugins.ec2.EC2Cloud
INFO: bootstrap()
Mar 12, 2024 3:23:29 PM hudson.plugins.ec2.EC2Cloud
INFO: Getting keypair...
Mar 12, 2024 3:23:29 PM hudson.plugins.ec2.EC2Cloud
INFO: Using private key jen4ec2-byadmin (SHA-1 fingerprint 1c:65:b0:6b:a9:2b:31:db:ab:57:db:5e:d6:b5:3d:c7:99:7c:cf:1d)

```

End Time:


```
Mar 12, 2024 3:29:13 PM hudson.plugins.ec2.EC2Cloud
INFO: Copying remoting.jar to: /tmp
Mar 12, 2024 3:29:13 PM hudson.plugins.ec2.EC2Cloud
INFO: Launching remoting agent (via Trilead SSH2 Connection):  java  -jar /tmp/remoting.jar -workDir /home/ec2-user
<===[JENKINS REMOTING CAPACITY]===>Remoting version: 3206.vb_15dcf73f6a_9
Launcher: EC2UnixLauncher
Communication Protocol: Standard in/out
This is a Unix agent
Agent successfully connected and online
```

Let's return to the nodes section by following this path:
Dashboard ➜ Manage Jenkins ➜ Nodes

"If you’ve followed the steps correctly, you’ll notice an additional drop-down field under ‘Nodes.’ Clicking on it will reveal something like ` ‘EC2 (name) – Description.’ ` In my case, it’s ` EC2(myec2)-ami custom made.` Depending on your EC2 cloud configuration choices, you might need to wait for a bit.




Click on it and you should see your ec2. Keep the AWS console open alongside Jenkins because although EC2 instances are created instantly, the time-consuming part is connecting to them. Here’s where I spent a lot of time googling and waiting. By default, Jenkins tries to connect to the EC2 instance’s private IP, not the public one. I emphasize this point so that you don’t have to go through the same experience." 


<img src="/part-3/pic-2/jp2-4.png" width="500" />

When you encounter the message:"This node is being launched. See log for more details"simply click on "See log for more details"


Do not be disheartened by messages similar to this one.

```
INFO: The instance EC2 (myec2) - ami custome made (i-08009777396f92f19) has a blank console. Maybe the console is yet not available. If enough time has passed, consider changing the key verification strategy or the AMI used by one printing out the host key in the instance console
Mar 13, 2024 7:53:02 AM hudson.plugins.ec2.EC2Cloud
INFO: The instance console is blank. Cannot check the key. The connection to EC2 (myec2) - ami custome made (i-08009777396f92f19) is not allowed
Mar 13, 2024 7:53:02 AM hudson.plugins.ec2.EC2Cloud
INFO: Failed to connect via ssh: There was a problem while connecting to 54.171.46.244:22

```
Navigate to your AWS console to locate your running EC2 instance. Once you confirm its status, you are close to completion; the primary focus now shifts to establishing a connection, which essentially involves waiting.


<img src="/part-3/pic-2/jp2-10.png" width="500" />

When you observe something akin to the content in the section below, it indicates that your instance is operational and connected, which allows us to utilize it as a node in our Jenkins environment.

```
INFO: Launching remoting agent (via Trilead SSH2 Connection):  java  -jar /tmp/remoting.jar -workDir /home/ec2-user
<===[JENKINS REMOTING CAPACITY]===>Remoting version: 3206.vb_15dcf73f6a_9
Launcher: EC2UnixLauncher
Communication Protocol: Standard in/out
This is a Unix agent
Agent successfully connected and online
```
A notable feature of this EC2 plugin that I appreciate is its ability to automatically terminate the instance if it remains unused for a period, effectively helping to save costs.

You may have to scroll up now and click on ‘Nodes’ next to ‘Dashboard’.

If you haven't made any changes to the "Disk Space Monitoring Thresholds," it means the default settings are applied by Jenkins. In that case, you might encounter an error related to the "Free Temp Space Threshold."

<img src="/part-3/pic-2/jp2-21.png" width="500" />



Delete this agent and simply return to the EC2 cloud settings on the Jenkins dashboard and configure the settings as per my recommendations.
Disk Space Monitoring Thresholds :  Tick
Free Temp Space Threshold :  0 
Free Temp Space Warning Threshold :  0


Once, I encountered a situation where adding a permanent agent to Jenkins required executing a command shown in a picture, which resolved the issue. However, this approach did not work when dealing with Amazon Linux 2.



<img src="/part-3/pic-2/jp2-12.png" width="500" />

After completing all the necessary steps, your Node page should resemble the example below.


<img src="/part-3/pic-2/jp2-14.png" width="500" />




Let’s take it for a spin.

Select an existing job or create a new one, then check the box labeled "Restrict where this project can be run." When you begin typing 'EC' in uppercase, you should observe your EC2 instance being populated.

“EC2 (myec2) - ami custome made (i-0ce453b6749f2f0a5) ”


Here is the outcome of my job.


```
Console Output
Started by user admin user
Running as SYSTEM
Building remotely on EC2 (myec2) - ami custome made (i-0ce453b6749f2f0a5) in workspace /home/ec2-user/workspace/test_job
[test_job] $ /bin/sh -xe /tmp/jenkins17277204568352314061.sh
+ echo Hello
Hello
+ cat /etc/os-release
NAME="Amazon Linux"
VERSION="2023"
ID="amzn"
ID_LIKE="fedora"
VERSION_ID="2023"
PLATFORM_ID="platform:al2023"
PRETTY_NAME="Amazon Linux 2023.3.20240304"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2023"
HOME_URL="https://aws.amazon.com/linux/amazon-linux-2023/"
DOCUMENTATION_URL="https://docs.aws.amazon.com/linux/"
SUPPORT_URL="https://aws.amazon.com/premiumsupport/"
BUG_REPORT_URL="https://github.com/amazonlinux/amazon-linux-2023"
VENDOR_NAME="AWS"
VENDOR_URL="https://aws.amazon.com/"
SUPPORT_END="2028-03-15"
Finished: SUCCESS
```

The shell code that was used in this job;

```
echo "Hello"
cat /etc/os-release
```

Thanks for sticking around. 
I trust you found it as enjoyable as I did.
Thank you.







