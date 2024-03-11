<img src="/part-1/pics/jenkins_log.png" width="100" />


# Exploring Jenkins:   A Comprehensive Lab
Welcome to this hands-on lab where we delve into the powerful combination of Jenkins and Docker. Our journey will be divided into multiple parts, each packed with practical insights and essential knowledge. Whether you’re a seasoned DevOps enthusiast or just starting out, this lab will provide valuable experience.

### Part 1: Setting Up Jenkins with Docker

 In the initial phase, we’ll install Jenkins using Docker. This dynamic duo allows us to create a flexible and scalable environment for continuous integration and continuous delivery (CI/CD). We’ll explore the seamless integration of Jenkins within Docker containers. Docker isn’t just a buzzword; it’s a game-changer! We’ll dive into some of Docker’s key features, such as containerization, image management, and networking. Additionally, we’ll configure Docker to optimize our Jenkins setup
 
 Once Jenkins is up and running, we’ll set up users, define roles, and explore role-based authentication. Understanding user management is crucial for maintaining a secure and collaborative environment. Jenkins thrives on its extensive plugin ecosystem. We’ll install essential plugins, explore their functionalities, and witness how they enhance our CI/CD pipelines. From Git integrations to build tools, we’ll cover it all.
 
  User-defined variables play a pivotal role in customizing Jenkins jobs. We’ll create jobs that utilize these variables, allowing for dynamic and adaptable workflows.Where does Jenkins store its configuration files? What about environment variables? We’ll uncover the secrets behind Jenkins’ file storage and environment management.
  Cron jobs are like the reliable clockwork of our system. We’ll explore how to schedule tasks at specific intervals using cron expressions. From periodic builds to cleanup routines, cron jobs keep our pipelines humming. We’ll learn how to trigger Jenkins jobs automatically using cron schedules. Whether it’s nightly builds, weekly reports, or monthly backups, we’ll harness the power of timed execution.
 
#### Lab Environment Details

- Host Machine: CentOS 8 
- Docker Version: 25.0.4 
- Jenkins Version: 2.448 
- User (sudo): orange
- My Working Directory: base

    
#### Prerequisites

- Basic knowledge of Linux and Docker 
- A similar lab environment (CentOS 8 with Docker) 



## Installing Docker:

To get started with Docker, you can follow the official installation instructions provided by Docker for CentOS.
https://docs.docker.com/engine/install/centos/


```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

```
sudo yum install -y yum-utils
```

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

We will not be making use of ‘docker compose’ command but let’s just install it anyway.

Let’s check the status of our docker, as you can see it's offline. Or at least it is in my case. Check it anyway.

```
sudo systemctl status docker
```


Let’s start it and make it persistent

```
sudo systemctl enable --now docker
```

Now lets check again

```
sudo systemctl status docker
```

![pic](/part-1/pics/jenkins-1.png)

To verify that our installation is fully functional, we’ll perform a quick check. 

```
sudo docker run hello-world
```

Evrything seems normal. However, if you enter the following command, an error will occur. 

```
docker images
```


To address this issue, I’ll add my user,‘orange’, to the group named ‘docker’, which was created during the installation process. You can follow a similar approach for your own user. 

```
sudo usermod -aG docker orange
```

![pic](/part-1/pics/jenkins-2.png)


To resolve this, exit out of the current session and log back in. If you try the command now, it should work as expected 

```
docker images
```



![pic](/part-1/pics/jenkins-3.png)



To keep things organized and tidy, I’ll create a separate directory for each of my Docker container. This way, I can place the Dockerfile within the corresponding directory. Additionally, if I ever need to mount a directory to my Docker container, I’ll use the same location. Clear separation and consistency make managing Docker images and containers much smoother. 
The directory ‘host_vol4jenkins’ inside ‘’ will be used to mount the Jenkins home directory

```
mkdir  jenkins_all
mkdir jenkins_all/host_vol4jenkins
cd jenkins_all/
vi Dockerfile
```

The Dockerfile's content;

```
FROM jenkins/jenkins

# Switch to the root user so we can install packages and create a user
USER root

# Install vim and iputils-ping
RUN apt-get update && apt-get install -y vim iputils-ping

# Create a user called 'jenkins-user' with root privileges
RUN useradd -m -s /bin/bash jenkins-user && echo "jenkins-user:pass11" | chpasswd

# Add 'jenkins-user' to the sudoers file so it has root privileges
RUN echo "jenkins-user ALL=(ALL:ALL) ALL" >> /etc/sudoers

# Switch back to the 'jenkins' user
USER jenkins
```

Let’s create a Docker image based on this Dockerfile. While you could simply pull an existing image, I prefer to customize it by adding a few tools. This way, if the need arises, I’ll have the necessary tools readily available 

Before building and running this Docker container, let’s create a custom network. This network will allow us to start multiple containers within the same network, facilitating seamless communication between them. 

```
docker network create net4jenkins
docker network ls
```

Time to build an image. Note the dot (.) meaning current directory.

 ```
docker -t jenkins .
```

Let’s confirm, you should see an image tagged ‘jenkins’
```
docker images
```

Before we embark on the journey of running the Jenkins Docker container, let’s ensure that all the necessary prerequisites are in place. We’ve already verified the existence of our network. Now, let’s double-check the required directories. You can do this by executing the following command: 

```
ls -l    jenkins_all/host_vol4jenkins
```

It’s time to launch our Jenkins Docker container and set sail on this DevOps adventure. 

```
docker run --name jenkins -v /home/orange/base/jenkins_all/host_vol4jenkins/:/var/jenkins_home -p 8090:8080 --restart=always  --network net4jenkins  -d jenkins
```

make sure the paths here are absolute otherswise  you might get the following error;


![pic](/part-1/pics/jenkins-4.png)

If you’ve already set up your Docker environment and are managing multiple containers, there’s a nifty command I find quite useful. It helps keep things tidy and provides a neat display of all running containers. Here’s the command: 

```
docker ps    --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"
```

 a neat display of all running and Stopped containers. Here’s the command: 

```
docker ps -a   --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"
```
## Welcome to the world of Jenkins! 

Given your setup, you can access your Jenkins instance using the following address:
Jenkins URL: http://209.97.128.146:8090
Since I was using DigitalOcean for my lab environment, the public IP address I provided (209.97.128.146) may not be applicable to your setup. Instead, you’ll likely use your local IP address to access Jenkins. Replace the IP address with your own if you’re running this locally. Happy Jenkins-ing!
```
http://209.97.128.146:8090
```

#### When accessing Jenkins, you’ll be prompted for a password. To retrieve this password, you have two options:
    1. Using Docker Logs:
        ◦ Run the following command to view the Jenkins logs:
          
```
docker logs jenkins
```

```


*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

b9a94debc33844f582c7ecc6ec915113

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
```
2. Directly from the Container:
      If you’re inside the Jenkins container, you can find the password at the following location:
```
/var/jenkins_home/secrets/initialAdminPassword
```

To get inside the Jenkins container;

```
docker exec -it jenkins bash
```

If you do not want to get inside the container, You can also execute the following on host machine to access the exact same file;

```
docker exec -it jenkins    cat /var/jenkins_home/secrets/initialAdminPassword
```


You can get it from the mounted directory on the host machine like so;

```
cat  /home/orange/base/jenkins_all/host_vol4jenkins/secrets/initialAdminPassword
```

[NOTE: the file will be deleted once password has been used but the password in logs will still be there.]

As you can see there are many ways to extract the password. Choose the method that suits your workflow, and you’ll have the Jenkins admin password at your fingertips! 

After entering the correct password, you’ll be prompted to make a choice. You can select from the following options:

##### 1. Install Suggested Plugins:
This option installs a set of recommended plugins that enhance Jenkins functionality right away
##### 2. Select Plugins to Install Later:
   If you prefer a more customized approach, you can choose this option. You’ll manually select the plugins you want to install later.
Make your selection based on your preferences.



<img src="/part-1/pics/jenkins-5.png" width="500" />



choose “install suggested plugins” unless you manually want to select and know what you are doing.



<img src="/part-1/pics/jenkins-6.png" width="500" />




After completing the installation, you’ll encounter the prompt to create the first admin user. Here are the fields you can fill in:

- Username: Choose a unique username for your Jenkins admin account.
- Password: Set a strong password to secure your account.
- Full Name: Enter your full name .
- Email Address: Provide an email address associated with this account.




<img src="/part-1/pics/jenkins-7.png" width="500" />



After this step, you will encounter the “Instance Configuration” screen. This screen displays the path to your Jenkins instance. If you prefer not to view the IP address, you can save this information to your localhost. It’s worth noting that I’ll be dismantling my lab shortly, so the IP address doesn’t concern me either way.



<img src="/part-1/pics/jenkins-8.png" width="500" />

## Creating First Job/Project:

In Jenkins, the terms Jobs and Projects are often used interchangeably. For the remainder of this lab, I’ll consistently refer to them as jobs.
Now, let’s proceed with creating a job according to the following steps:

#### test_job

To create a new job in Jenkins, follow these steps:
    1. Click on “Dashboard”.
    2. Select “New Item”.
    3. In the “Enter an item name” field, type any desired name. For example, I’ll use “test_job”.
    4. Finally, click “OK” to create the job.

Let’s walk through the steps to create a Jenkins job:

    1.  Click on“Dashboard”.
    2.  Select“New Item”.
    3.  In the“Enter an item name” field, type any desired name. For example, I’ll use “test_job”.
    4.  Finally, click“OK”to create the job.

Next, follow these steps to configure the job:
    1.  Scroll down to “Build Steps”and choose “Execute Shell”.
    2.  In the “Command” section, enter the following shell command:

```
echo "Hello"
```


3. Click on “Save”.
       4. Now, click on “Build Now”.

Soon, you should see the build number appearing. If it’s the first build, it will be labeled as “1”. Keep an eye on the “Build History” section



<img src="/part-1/pics/jenkins-9.png" width="250" />


## Installing plugins

Before proceeding, let's install some plugins. Each enterprise has unique needs, so not everyone requires all the plugins. Jenkins offers the option to choose from "1- Install Suggested Plugins" or "2- Select Plugins To Install," which we encountered during the installation of Jenkins. Based on my plan/requirements, I will select a few plugins now. I will install multiple plugins at once because Jenkins needs to be restarted after each plugin, and I already know what I need. So, here are the plugins I have chosen for the time being:

#### 1- Pipeline: AWS Steps



<img src="/part-1/pics/jenkins-13.png" width="250" />

To install the “Pipeline: AWS Steps” plugin in Jenkins, follow these steps:
    1. Navigate to the “Dashboard”.
    2. Click on “Manage Jenkins”.
    3. Within the management section, select “Plugins”.
    4. Choose “Available plugins”.
    5. In the search bar, type “Pipeline: AWS Steps” and check the checkbox next to it.
    6. If you plan to install additional plugins, you can go back to “Available plugins” and repeat the same procedure.




<img src="/part-1/pics/jenkins-12.png" width="250" />

The hyperlink to this step will be;
http://209.97.128.146:8090/manage/pluginManager/available

You can copy the ‘/manage/pluginManager/available’ from this part of the URL and append it to your Jenkins address—whether that’s a DNS name you’ve configured or simply an IP:Port, similar to my setup. 
       
    1. If you want to check whether you already have certain plug-ins, search in the search bar after visiting this URL: 
       http://209.97.128.146:8090/manage/pluginManager/installed
       

#### 2- S3 publisher plugin
Repeat the steps outlined in the previous section titled ‘1- Pipeline: AWS Steps.’ However, this time, your task is to search for the term ‘S3 publisher plugin’ that’s all.



<img src="/part-1/pics/jenkins-14.png" width="250" />

#### 3-  SSH
Repeat the steps outlined in the previous section titled ‘1- Pipeline: AWS Steps.’ However, this time, your task is to search for the  ‘SSH’. You will see warning message regarding using this plugin. As this is my lab environment I am going to go ahead with it.



<img src="/part-1/pics/jenkins-15.png" width="250" />

#### 4-  Docker
Repeat the steps outlined in the previous section titled ‘1- Pipeline: AWS Steps.’ However, this time, your task is to search for term ‘Docker’.



<img src="/part-1/pics/jenkins-16.png" width="250" />

#### 5-  AnsiColor
Repeat the steps outlined in the previous section titled ‘1- Pipeline: AWS Steps.’ However, this time, your task is to search for term ‘AnsiColor’. If you fancy colourful output in your ‘Console Output ’ then you can install this, but I am not going to. 



<img src="/part-1/pics/jenkins-17.png" width="250" />

#### 6- Role-based Authorization Strategy
_In this part of the lab, this Plugin is Required_

All the steps are same as above in “1- Pipeline: AWS Steps”. This time you will have to search for the term ‘Role-based Authorization Strategy’.



<img src="/part-1/pics/jenkins-24.png" width="250" />

After finish installing all the plugin, If you can still see the check box next to 
_‘Restart Jenkins when installation is complete and no jobs are running’_   check that box,




<img src="/part-1/pics/jenkins-44.png" width="250" />

 
 
 Otherwise click on “Download Progress” in the left side of the screen and you should see check box there ;
 





<img src="/part-1/pics/jenkins-18.png" width="250" />


You can check this box and Jenkins will start to reload, 





![pic](/part-1/pics/jenkins-19.png)


After reconnecting, you may be asked to enter your password again. 







<img src="/part-1/pics/jenkins-20.png" width="300" />







## Role-based Authentication:

We will be evaluating the functionalities of the "Role-based Authorization Strategy" plugin. Initially, we need to enable this feature in our Jenkins. To accomplish this, navigate to the 'Dashboard,' then select 'Manage Jenkins.' Once there, click on 'Security.'



<img src="/part-1/pics/jenkins-27.png" width="300" />


Under ‘Authorization ’ dropdown box, choose ‘Role-based Strategy’




<img src="/part-1/pics/jenkins-26.png" width="300" />





After completing this setup, we will proceed to create three additional jobs. Since you are already familiar with the job creation process, having created one in the previous step, I will simply list their names and the contents of the shell section. These remaining three jobs are identical, with the only difference being the shell command. While these jobs may not be elaborate, their purpose will become clear once you have finished this section.



##### test_job2
```
echo "This is Test 2"
```
##### dev_job1
```
echo "This is Job is for Dev 1"
```
##### dev_job2
```
echo "This is Job is for Dev 2"
```

we can see all these jobs on our main Dashbaord.



<img src="/part-1/pics/jenkins-23.png" width="300" />


### Create a User:

Navigate to the 'Dashboard,' then select 'Manage Jenkins.' Once there, click on 'Users,' and then choose 'Create User.' Name the user "view_tests" and complete all the remaining fields as shown in the figure below.



<img src="/part-1/pics/jenkins-25.png" width="300" />


### Create a Role:
Navigate to the 'Dashboard,' then select 'Manage Jenkins.' Once there, click on 'Manage and Assign Roles.' If you do not see this option, ensure that you have already installed the necessary plugin.



### Create a Global Role:
Under 'Global roles,' input 'viewonlyrole' in the 'Role to add' field, and then click on 'Add.' After doing so, select the 'Read' box next to the newly created 'viewonlyrole' in the 'Overall' section as shown in the picture.



<img src="/part-1/pics/jenkins-28.png" width="300" />



"We will only check this checkbox in 'Global roles' as we will further fine-tune it. Proceed to the section labeled 'Item roles.'"
Create an Item Role:
In the 'Item roles' section, enter 'test_.*' (without single quotes) in the 'Pattern' field, and name it 'viewonly_itemrole' in the 'Role to add' field. It should now appear in the 'Item roles' section. Check only the 'Read' and 'Build' boxes in the 'Job' section.

You can see the outcome of your pattern by clicking on the pattern ‘test_.*’ . It is highlighted  in blue and it’s the pattern you have specified earlier. Once happy with the results, click save.




<img src="/part-1/pics/jenkins-29.png" width="300" />



## Create Role Assignment:

In this final step, we consolidate user, role, and item roles.

Navigate to “Global roles”.
Click “Add User” and enter the username (in this case, it’s “view_tests”).
Note that if your username contains hyphens or underscores, Jenkins will display them as empty spaces, which can be confusing.
However, if you enter an invalid username, Jenkins will strike it through to indicate that it’s not valid. Feel free to test this by entering a non-existent username and observe the strikethrough effect.
Proceed with the configuration, and remember to save your changes!



<img src="/part-1/pics/jenkins-30.png" width="300" />




<img src="/part-1/pics/jenkins-31.png" width="300" />




If you’d like to stay logged in with your main account and test a separate user in a different session, consider using a completely different browser. Private browsing in Firefox doesn’t fully isolate sessions. To achieve true separation, open a different browser and navigate to the following link: 

http://209.97.128.146:8090

Enter name and credentials for user “view_tests”. Now you should only see two jobs starting with the name ‘test_’ as this is how we configured it to behave.



<img src="/part-1/pics/jenkins-32.png" width="300" />


Since we’ve allowed the “Build” operation, you now have the option not only to view jobs starting with “test_” but also to actively build them. Feel free to proceed with your desired actions 



<img src="/part-1/pics/jenkins-34.png" width="300" />




## Environment Variables:

To define variables in Jenkins and utilize its built-in environment variables, follow these steps:

1. Jenkins Official Documentation:
  You can explore the complete list of environment variables accessible within Jenkins by visiting the official documentation:
https://www.jenkins.io/doc/pipeline/tour/environment/

2. Local Link on Your Jenkins Machine:
   Alternatively, you can directly access the environment variables list on your Jenkins instance by clicking this link: Environment Variables in Jenkins

http://209.97.128.146:8090/env-vars.html/
[ change the ip:port ]
       
Within Build Steps:
While configuring your build steps, you’ll find a hyperlink labeled “See the list of available environment variables” (as shown in the image below).
Click on it to explore the available variables and make informed choices for your Jenkins setup.
Feel free to explore and utilize these variables as needed! 



<img src="/part-1/pics/jenkins-37.png" width="300" />




Now that we’ve covered the basics, let’s proceed with the exercise. In this task, we’ll utilize a variable we define ourselves and combine it with one of Jenkins’ environment variables. To get started, let’s create a new job 


- Navigate to the Dashboard in Jenkins.
- Click on “New Item”.
- In the field labeled “Enter an item name,” provide any desired name (for example, I’ll use “job_with_param”).
- Confirm the name by clicking “OK.”
- Check the checkbox next to “This project is parameterized?”
- Click on “Add Parameter” and select “String Parameter.”




<img src="/part-1/pics/jenkins-35.png" width="300" />




In the Name enter ‘Location_jen’ , Default value ‘London’.
Scroll down to ‘Build Steps ’ 



<img src="/part-1/pics/jenkins-36.png" width="300" />





Enter the following code in ‘Command’ Section and click Save.

```
echo "***********************************************************"
echo "Our Pre-defined Var called Location_jen Default Value is: $Location_jen"
echo "Your Worspace path for Jenkins is: $WORKSPACE"
echo "***********************************************************"
```



<img src="/part-1/pics/jenkins-38.png" width="300" />



## What Goes Where:

In this exercise, we’ll explore where Jenkins stores files when we don’t specify a specific location.

- Locate an existing job (for example, let’s use the job named “test_job2”).
- Navigate to the “Command” section within the job configuration.
- Enter the following code snippet.
- Finally, click “Save” to apply the changes.

```
echo "this is test 2" >> /tmp/findfile1.txt
echo "this is test 2" >> findfile2.txt
echo "this is test 2" >> $HOME/findfile3.txt
mkdir find2
echo "this is test 2" >> find2/findfile.txt
```




<img src="/part-1/pics/jenkins-39.png" width="300" />


The Jenkins Home Directory (often referred to as JENKINS_HOME) is a crucial location where Jenkins stores all its configuration data. When you install Jenkins, it automatically creates this directory. By default on linux-based systems, it resides under ~/.jenkins. Within this directory, you’ll find various files and subdirectories that hold essential information for Jenkins to operate effectively.

From the Picture below everything should be clear I have highlighted the files and directories. You can see where each file was created. You should observer similar behaviour on your machine too.
The key takeaway here is that Jenkins keeps everything within its own home directory.

Notably, the “tmp” directory used by Jenkins during our building a job/project is specific to Jenkins itself and not the host machine. If you run the command ls /tmp, on your host machine you won’t see any of the files or directories related to our Jenkins job. 
Remember, Jenkins manages its data within its designated home directory, ensuring proper isolation from the host environment
 The rest is pretty much clear.


<img src="/part-1/pics/jenkins-40.png" width="800" />


Another way to observe this job will be to go into the docker that Jenkins is installed on. And observe all these commands over there like in picture below;



<img src="/part-1/pics/jenkins-41.png" width="800" />


## Build Job using cron:
If you need your job to run at specific times, consider using cron. This feature in Jenkins allows you to schedule tasks precisely according to your requirements. 

- Navigate to the Dashboard in Jenkins.
- Click on the “test_job”. You can pick any other job you have.
- Next, click on “Configure”.
- Configure Build Triggers:
- Scroll down to the “Build Triggers” section.
- Check the box next to “Build periodically?”.
- In the “Schedule” field, enter the following:

```
* * * * *
```



(This runs the job every minute.)
Save Your Changes:
Click “Save” to apply the configuration.
Note that running it every minute might seem extreme, but in a lab environment, every minute feels like an hour! :)




<img src="/part-1/pics/jenkins-42.png" width="300" />




Keep an eye on build history. 



<img src="/part-1/pics/jenkins-43.png" width="300" />



_To experiment with or find the cron schedule expression you can use an online utility at_ https://crontab.guru/



Once happy with the results. Go back into ‘Configure’ and just un-check the box next to ‘Build periodically? ’ to disable it. Or just delete the job entirely.

###### And That’s All.
###### Thanks.











