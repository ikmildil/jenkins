# How to Create Different Agent Nodes in Jenkins

In this section, we will delve into Jenkins Node options, focusing on Permanent agents. I will utilize two CentOS containers of different versions, 7 and 8, to highlight the configuration variances and tweaks required based on the OS and its specific version. Initially, I will demonstrate the setup with a CentOS 8 container followed by CentOS 7, adding and running jobs on each accordingly. While this segment shouldn't be challenging, I encountered some surprises that I would like to share with you, leading me to create a post about it.

When it comes to testing a Dockerfile or crafting your own, my suggestion is to start small. Begin by creating a container without adding any components, build and run it, then enter the container to install applications and address any errors encountered. Install the necessary apps, document the commands used, transfer them to a Dockerfile, and build an image from it. Avoid including passwords in this process; although I have done so in my home lab for convenience, it is not considered best practice.

When revisiting older posts or exercises, unexpected challenges may arise due to evolving technologies. To mitigate this, strip away unnecessary options from the container and gradually add components step by step. This approach is why I am presenting two versions of CentOS with distinct configurations. Without further ado, let's delve into building images and creating containers.

The subsequent commands are not essential for this lab, but I’m including them to demonstrate my setup.

```
[orange@centos8 base]$ ls
centos7_all  centos8_all  jenkins_all

[orange@centos8 centos8_all]$ ls
Dockerfile  Dockerfile.old  key4centos8  key4centos8.pub


[orange@centos8 centos7_all]$ ls
Dockerfile  key4centos8  key4centos8.pub

docker images --filter "dangling=true"

docker rmi $(docker images -q --filter "dangling=true")

docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"

[orange@centos8 centos8_all]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED        STATUS       PORTS                                                  NAMES
4cc481eeedd8   jenkins   "/usr/bin/tini -- /u…"   45 hours ago   Up 3 hours   50000/tcp, 0.0.0.0:8090->8080/tcp, :::8090->8080/tcp   jenkins



[orange@centos8 centos8_all]$ docker network ls
NETWORK ID     NAME          DRIVER    SCOPE
4a19e353878a   bridge        bridge    local
fc2d2b2b38b8   host          host      local
94eea1a8a503   net4jenkins   bridge    local
56bd2986823c   none          null      local

```

### Creating a Key Pair:

I have not utilized this key; it is merely for illustrative purposes to demonstrate how they can be created. You can infer the key names from the commands I have previously shown and from the Dockerfile. These names may vary. Before proceeding with this lab, ensure you have a pair of keys - both public and private - and name them appropriately. While the names are not crucial, they aid in recognizing them.

```
[orange@centos8 temp-keys]$ ssh-keygen -f keys4dockers

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):

Enter same passphrase again:

Your identification has been saved in keys4dockers.
Your public key has been saved in keys4dockers.pub.
The key fingerprint is:

SHA256:XmfWCMoh+f+NG2MnGq1gqw9OrjPafkQ8xOhPTaIDjt0 orange@centos8
The key's randomart image is:
+---[RSA 3072]----+
|     o           |
|  . . +..        |
| + + +o+. .      |
|. o E =+.o . o   |
|     = .S . = .  |
|      o. o =     |
|     .o + o * .  |
|   .o+.o o * B   |
|  .o+=+oo o +..  |
+----[SHA256]-----+
[orange@centos8 temp-keys]$ ls
keys4dockers  keys4dockers.pub

```



## Adding Permanent Agent [CentOS 8]

Within this section, we will embark on constructing an image of CentOS 8 followed by the creation of a container derived from it. This process involves compiling the necessary dependencies, configuring the environment, and optimizing the image for efficient containerization. 

We will generate a Docker container using the provided Dockerfile for the CentOS operating system. This Dockerfile is based on CentOS 8 to be specific and performs the following actions:

Modifies the CentOS repository configuration to use a specific base URL.
Installs the openssh-server package.
Installs Java as a prerequisite for integration with Jenkins.
Creates a user 'cent1-user' with root privileges and sets up SSH access using an authorized key.
Configures SSH settings and starts the SSH daemon as the container's main command.

```
RUN rm -rf /run/nologin
```

What this action accomplishes is the removal of the /run/nologin file to enable SSH login. 
This file typically displays the message "System is booting up," which persists even after systemd installation, potentially hindering SSH access. 
Initially, the error messages were not particularly informative, leading to confusion and unnecessary troubleshooting regarding Docker security, network settings, usernames, and passwords. 
Despite not requiring SSH access due to using a public key for authentication, this issue caused significant time wastage and frustration. 
And I thought I should share that with you. Let’s carry on.

You should find a Dockerfile for my CentOS 8 setup.


```
FROM centos:8

RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

RUN yum -y install openssh-server

# Since I will be integrating this into the Jenkins Node, Java is a prerequisite for this setup.
RUN yum -y install wget && \
    cd ~ && \
    wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.rpm && \
    yum -y install ./jdk-17_linux-x64_bin.rpm

# Create user cent1-user with root privileges and create home directory

# WARNING: THIS Dockerfile INCLUDES CREDENTIALS::::: BELOW:::::


RUN useradd -m cent1-user && echo cent1-user:pass11 | chpasswd && usermod -aG wheel cent1-user

RUN mkdir /home/cent1-user/.ssh && \
    chmod 700 /home/cent1-user/.ssh

COPY key4centos8.pub /home/cent1-user/.ssh/authorized_keys

RUN chown cent1-user:cent1-user   -R /home/cent1-user && \
    chmod 400 /home/cent1-user/.ssh/authorized_keys

RUN ssh-keygen -A
# very important to rm this file otherwise you will not be able to ssh into it.
RUN rm -rf /run/nologin


CMD /usr/sbin/sshd -D

```

Let's make sure we have everything we need.

```
[orange@centos8 centos8_all]$ ls
Dockerfile  Dockerfile.old  key4centos8  key4centos8.pub
```

Time to Build an image.

```

[orange@centos8 temp-keys]$ docker build -t centos8   .

```

The following command will create docker name 'cent1' out of this image.

```

[orange@centos8 centos8_all]$ docker run --name cent1 --network net4jenkins  -d centos8
```


When you have numerous Docker containers running, and the screen becomes cluttered after running the ` docker ps ` command, I discovered the following command to be quite useful in such situations.


```
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"
```

Before proceeding to the Jenkins dashboard, let's review the remaining settings.

```

[orange@centos8 temp-keys]$ docker inspect cent1 | grep -i ipaddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.3",



[orange@centos8 temp-keys]$ docker inspect cent1 | grep -i network
            "NetworkMode": "net4jenkins",
        "NetworkSettings": {
            "Networks": {
                    "NetworkID": "94eea1a8a503bf63dc1e08fa21ed628a706267106f3aa472f3f9d918d0dabfe4",

```

Take note of the private key as it will be required in the Jenkins dashboard.

### Create Credential:
  Prior to configuring a node in Jenkins, it is essential to first establish the necessary credentials, as they will be needed during the node setup process.

In the Jenkins dashboard, follow the path.

Dashboard  ➜ Manage ➜ Jenkins     ➜ Credentials     ➜   Global



I will outline the settings I have adjusted, while leaving the rest at their default values. I will recommended to follow a similar pattern when configuring your setup.

- click ‘Add Credential’ 
- Kind: SSH Username with Private Key
- Scope: global
- ID: ssh_private_key_centos8_cent1-user
- Description  : ssh_private_key_centos8_cent1-user Des
- username: cent1-user
- Under Private Key click  “Enter directly here” You will be required to paste your private key (key4centos8 in my case), which can be obtained by:

```
[orange@centos8 centos8_all]$ vi key4centos8
[orange@centos8 centos8_all]$ cat key4centos8
```

If no passphrase has been set, please leave it empty.


<img src="/part-2/pic-2/jp2-28.png" width="500" />

### Create Agent Node:

To create an agent node in Jenkins, follow path;
Dashboard  ➜ Manage Jenkins     ➜ Nodes

I will outline the settings I have adjusted, while leaving the rest at their default values. I will recommended to follow a similar pattern when configuring your setup.

Name :node1_centos8_cent1_user
Description :Node of type centos8 for user cent1-user
Number of executors :1
Remote root directory :/home/cent1-user
Labels :node1_centos8_cent1_user_label
Launch method : Launch agents via SSH
Host : 172.18.0.3   
Alternatively, enter 'cent1'. This is one of the reasons we established a network and ensured that  docker containers are all within the same network.

<img src="/part-2/pic-2/jp2-29.png" width="500" />

<img src="/part-2/pic-2/jp2-21.png" width="500" />


Credentials :  ssh_private_key_centos8_cent1-user  the one we create above 
Host Key Verification Strategy : Manually trusted key Verification Strategy.
The rest: leave default

because we chose “Manually trusted key Verification Strategy” sometimes you do have to press on the ‘Trust SSH Host Key’ shown in picture if not you should see it all connected.


<img src="/part-2/pic-2/jp2-25.png" width="250" />


<img src="/part-2/pic-2/jp2-26.png" width="500" />


Now, let's execute a job. If you have been following from part-1, you should already have several jobs displayed on your Jenkins dashboard. If not, you can create one. If you are unsure how to proceed, refer back to that section for guidance.

Go to dashboard    ➜ pick a job (in my casse test_job)    ➜ configure (if using existing job)     ➜check “Restrict where this project can be run? ” 

When entering a label in the 'Label Expression' field, start typing the label. Sometimes, after typing a few letters, the label may appear, but not always. I will suggest label if it does not popup to avoid errors rather than typing it out. Additionally, even if the label pops up when clicked, it may append a space and display warnings such as; [this is from my setup.]

```
No agent/cloud matches this label expression. Did you mean 'node1_centos8_cent1_user' instead of 'node1_centos8_cent1_use'?
```
In such cases, simply delete the space and proceed to click 'Save' regardless.
The shell code used for this job is;
```
echo "Hello"
cat /etc/os-release
```


```
Console Output
Started by user admin user
Running as SYSTEM
Building remotely on node1_centos8_cent1_user (node1_centos8_cent1_user_label) in workspace /home/cent1-user/workspace/test_job
[test_job] $ /bin/sh -xe /tmp/jenkins16061293793311013520.sh
+ echo Hello
Hello
+ cat /etc/os-release
NAME="CentOS Linux"
VERSION="8"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="8"
PLATFORM_ID="platform:el8"
PRETTY_NAME="CentOS Linux 8"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:8"
HOME_URL="https://centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"
CENTOS_MANTISBT_PROJECT="CentOS-8"
CENTOS_MANTISBT_PROJECT_VERSION="8"
Finished: SUCCESS
```


## Adding Permanent Agent [CentOS 7]

To create a Docker image for CentOS 7, you can follow a similar process to the one outlined for CentOS 8. Begin by ensuring that you have a 64-bit CentOS 7.

Utilize the provided Dockerfile for execution, ensuring you are in the correct directory. We have established two separate folders, each containing copies of the same key but with distinct Dockerfiles. These files feature unique configurations and utilize different usernames.

In this Dockerfile we are essentially using CentOS 7. It installs the openssh-server package, then proceeds to install Java 17 by downloading and installing the JDK package. It creates a user named 'cent2-user' with a password, sets up SSH access using an authorized key, and configures permissions for the user's SSH directory. Additionally, it generates SSH keys and starts the SSH daemon as the main command.
In this section, I aim to emphasize the varied configurations and setups influenced not only by the OS version but also by the individual or organizational choices when creating containers. These containers are essentially customized by their creators, allowing them to select specific configurations and components. While I won't delve deeply into the setup process, the remaining procedures remain largely the same with minor adjustments that I have outlined below. 

So lets start by building the image.

```
[orange@centos8 centos7_all]$ #docker build -t centos7 .
```

The Dockerfile for centos7

```
FROM centos:7

RUN yum -y install openssh-server

RUN yum -y install wget && \
    cd ~ && \
    wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.rpm && \
    yum -y install ./jdk-17_linux-x64_bin.rpm

RUN useradd cent2-user && \
    echo "pass" | passwd cent2-user  --stdin && \
    mkdir /home/cent2-user/.ssh && \
    chmod 700 /home/cent2-user/.ssh

COPY key4centos8.pub /home/cent2-user/.ssh/authorized_keys

RUN chown cent2-user:cent2-user   -R /home/cent2-user && \
    chmod 400 /home/cent2-user/.ssh/authorized_keys

RUN ssh-keygen -A


#RUN yum -y install epel-release && \
#    yum -y install python3-pip && \
#    pip3 install --upgrade pip

CMD /usr/sbin/sshd -D
```

```
[orange@centos8 centos8_all]$ docker run --name cent2 --network net4jenkins  -d centos7
```



<img src="/part-2/pic-2/jp2-2.png" width="500" />

The steps here mirror those for CentOS 8. When in the Jenkins Console:
- Create a separate credential for this node. Even though the same private key is used, ensure distinct users are set up accordingly.
- After creating the node, navigate to  Dashboard  ➜ Manage Jenkins     ➜ Nodes
- Click on the name of the node in my case “node2_centos7_cent2_user”

Monitor the Dashboard for the appearance of the 'Trust SSH Host Key' option as depicted in the image. 
If you spot it, click on it; otherwise, you should observe that everything is connected.



<img src="/part-2/pic-2/jp2-25.png" width="250" />


<img src="/part-2/pic-2/jp2-21.png" width="500" />


Now, let's execute a job. Below is my result for this Agent Node of CentOS 7.

```
Console Output
Started by user admin user
Running as SYSTEM
Building remotely on node2_centos7_cent2_user (node2_centos7_cent2_user_label) in workspace /home/cent2-user/workspace/test_job
[test_job] $ /bin/sh -xe /tmp/jenkins9299795059514948884.sh
+ echo Hello
Hello
+ cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

Finished: SUCCESS
```

You have the option to compare the outcomes.



If you wish to access the job's code, it is provided here.

```
echo "Hello"
cat /etc/os-release
```

Some useful commands you can try out;

```
[orange@centos8 centos8_all]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                                  NAMES
7cfb651d82a8   centos7   "/bin/sh -c '/usr/sb…"   6 seconds ago    Up 4 seconds                                                           cent2
cb6969a7f18e   centos8   "/bin/sh -c '/usr/sb…"   24 seconds ago   Up 23 seconds                                                          cent1
4cc481eeedd8   jenkins   "/usr/bin/tini -- /u…"   45 hours ago     Up 3 hours      50000/tcp, 0.0.0.0:8090->8080/tcp, :::8090->8080/tcp   jenkins
```

You have the ability to ping your Docker containers without accessing them directly, allowing you to remain on your host system. 
This method can help save time and streamline your workflow.

```
[orange@centos8 centos8_all]$ docker exec -it jenkins ping -c2 cent1
PING cent1 (172.18.0.3) 56(84) bytes of data.
64 bytes from cent1.net4jenkins (172.18.0.3): icmp_seq=1 ttl=64 time=0.095 ms
64 bytes from cent1.net4jenkins (172.18.0.3): icmp_seq=2 ttl=64 time=0.090 ms
```

```

--- cent1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1058ms
rtt min/avg/max/mdev = 0.090/0.092/0.095/0.002 ms
[orange@centos8 centos8_all]$
[orange@centos8 centos8_all]$ docker exec -it jenkins ping -c2 cent2
PING cent2 (172.18.0.4) 56(84) bytes of data.
64 bytes from cent2.net4jenkins (172.18.0.4): icmp_seq=1 ttl=64 time=0.092 ms
64 bytes from cent2.net4jenkins (172.18.0.4): icmp_seq=2 ttl=64 time=0.094 ms

```

```

--- cent2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1063ms
rtt min/avg/max/mdev = 0.092/0.093/0.094/0.001 ms

```
```
[orange@centos8 centos8_all]$ docker inspect cent1 | grep -i ipaddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.3",
```


```
[orange@centos8 centos8_all]$ docker inspect cent2 | grep -i ipaddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.4",
```


```
[orange@centos8 centos8_all]$ docker inspect jenkins | grep -i ipaddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.2",
```
