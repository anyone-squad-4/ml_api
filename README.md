
![](https://web-api.textin.com/ocr_image/external/139b1b0b41501131.jpg)

# Deploying a project to AWS

This document provides a set of instructions for deploying a project to AWS EC2 instances (virtual machines). It is divided into three tables (checklists):

1.Launching an EC2 instance, connecting to it via SSH (and turning it off).

2.Installing docker and docker-compose within the EC2 instance, creating the necessary Docker images, transferring the project’s code and executing it.

3.Create a database instance (using the RDS service from AWS), connect to it using DBeaver (from your local machine) and write to/read from it, from your EC2 machine.

# 1.Launch an EC2 instance and connect to it

Important: Steps 1-6 and 9-10 are needed only if you are using your own AWS account, and they need to be run from the AWS console: go to the EC2 section and click on Launch Instance, then follow these instructions. Remember that you are NOT required to create a personal account, this is optional and you do it under your own risk of incurring expenses. If you decide to use the resources provided to you through AnyoneAI’s AWS account, simply run steps 7-8 with the public IP address and PEM key provided by the instructor (to be found on the Discord chat of your Squad).


| No.  | Step  | Description  | Comments  |
| --- | --- | --- | --- |
| 1* | Select AMI  | Select the image of the machine you want to launch, which already contains a working operating system. We will be using the “Ubuntu” AMI.  | The commands that install Docker work only for Ubuntu, so make sure to use this AMI and not other Linux ones! |
| 2* | Select machine (instance type)  | Set the t2.micro, a small machine with only 1GB of memory.  |  |
| 3* | Select or create key file and set the appropriate permissions  | Create a .pem file and save it into a known location. Set only-read permissions by running chmod 400 &lt;PEM_FILE&gt; | You will need this path later for step 8.  |
| 4* | Configure security group  | Allow traffic to everyone by using 0.0.0.0 (default) or to your own public IP address, for the following ports: 8000, 9000, 5432.  | 8000: for Rest API 9000: for UI 5432: for PostgreSQL  |
| 5* | Configure storage  | Default will be 8GB, however this is insufficient. Increase it to 30GB.  | You can use up to 30GB for free.  |
| 6* | Launch instance  | Press “Launch instance” and go to the instances panel (for example, go to “EC2dashboard” and then click on “Instances”.) |  |
| 7  | Download and install SSH client  | On Linux and MacOS, there’s nothing to be done. On Windows, you can install, for instance, OpenSSH.  | Other options (on Windows) are PuTTY or MobaXterm.  |
| 7.5  | Modify/create your SSH config file (optional)  | Host anyone HostName &lt;PUBLIC_ADDRESS&gt;User ubuntu IdentityFile &lt;PATH_TO_PEM&gt;IdentitiesOnly yes Example:Host anyone HostName &lt;PUBLIC_Aec2-3-147-8-28.us-east-2.compute.amazonaws.comUser ubuntu IdentityFile \~/Downloads/anyone3.pemIdentitiesOnly yes  | If you do this, you can simply connect to the instance by typing ssh anyone. You will need to modify the config file each time you launch a new instance (because the IP changes).  |
| 8  | Connect to EC2 instance  | Use your SSH client to connect to the machine. The user is ubuntu (unless you are using a different AMI, in which case it might change, however note that the instructions here imply an Ubuntu instance). The full command would be ssh -i &lt;PATH_TO_PEM&gt; ubuntu@&lt;PUBLIC_ADDRESS&gt;If you have done step 7.5 you can simply run this instead:ssh anyone  | The PEM file location needs to be replaced, as well as the machine’saddress (“PublicIPv4 DNS” in the dashboard)  |
| 9* | Terminate instance (once you are done)  | On the dashboard, go to “Instance State” and then “Terminate”. |  |
| 10* | Set an alarm (optional but recommended)  | This will notify you if you have a running instance that hasn’t been used in a while, for example in case you forget to terminate it (previous step).  | This is important as you only have 750 hours of free usage.  |


<!--  -->

*Required only if you are using a personal account (instead of the EC2 instance provided by AnyoneAI)

# 2. Install docker/docker-compose and run project

This has to be done within the EC2 instance, once you are connected via SSH.


| No.  | Step  | Description  | Comment  |
| --- | --- | --- | --- |
| 1  | Install Docker  | sudo apt-get update sudo apt-get upgrade -y sudo apt-get install apt-transport-https ca-certificates curl software-properties-common -y curl -fsSL https://download.docker.com/linux/ubuntu/gpg |sudo apt-key add -sudo add-apt-repository "deb [arch=amd64]https://download.docker.com/linux/ubuntu &#36;(lsb_release -cs) stable" sudo apt-get update sudo apt-get install docker-ce -y  | You will be prompted to press Enter at some point.  |
| 2  | Initiate Docker  | sudo systemctl start docker sudo systemctl enable docker sudo systemctl status docker  |  |
| 3  | Add the user ubuntu to the docker group  | Run the following command to add the user to the group:sudo usermod -aG docker ubuntu Log out of the EC2 instance (that is, end your SSH session), then connect back and run groups to see if the user was added correctly. You should see that the last group is docker.  | The groups command tells you which groups the current user belongs to.  |
| 4  | Install Docker Compose  | sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-&#36;(uname -s)-&#36;(uname -m)" -o /usr/local/bin/docker-composesudo chmod +x /usr/local/bin/docker-composedocker-compose --version  |  |
| 5  | Copy project to EC2 instance  | Download the project from Academy: “Microservice architectures - Theory Support Code”. Create a folder called ml_model_deployment, move ml_app_microservices.zip there and unzip it. Copy it to the EC2 instance using the following command:scp -i &lt;PATH_TO_PEM&gt; -r path/to/ml_model_deploymentubuntu@&lt;PUBLIC_ADDRESS&gt;:or, if you did 7.5 from Table 1, simply run the following command instead:scp -r path/to/ml_model_deployment anyone: | Notice the colon (:)in the end. It’simportant! This will copy the file into the home folder of the ubuntu user (/home/ubuntu).Also, notice the -r option. It is used to copy folders (-r stands for “recursive”as the contents of the folder are explored recursively and transferred to the destiny)  |
| 6  | Adjust permissions for Docker socket  | sudo chmod 666 /var/run/docker.sock |  |
| 7  | Run docker-compo se  | cd ml_model_deployment/ml_api_microservicesdocker-compose up  |  |
| 8  | Create SSH tunnel? | ssh -L 8000:localhost:8000 |  |


<!--  -->

# 3. Create an instance in RDS and write/read from it

As in Table 1, step 1 is only needed if you are launching the resources from your own personal account.


![](https://web-api.textin.com/ocr_image/external/3d7f68135215ed7e.jpg)


| No.  | Step  | Description  | Comment  |
| --- | --- | --- | --- |
| 1* | Create RDS instance  | Go to RDS / Create database, and use the :-Standard Create -Engine Type: PostgreSQL -Engine Version: PostgreSQL 16.3 R2 -Template: Free Tier -DB instance identifier: prueba (or whichever name you like) -Master username: postgres -Master password: ******-Instance configuration: db.t3.micro -Storage type/size: General Purpose SSD / 20GB -Connectivity:-Connect to your EC2 instance. -VPC: default -Public Access: Yes -VPC security groups: default -Availability Zone: no preference -Database authentication: Password authentication -Database options -Initial database name: anyone Wait until the instance is launched and copy the host name.  | Take note of the username and password. Also of the host name once the instance is launched. You will need them in the next steps.  |
| 2  | Test connection using DBeaver  | Go to Database / New Database Connection. Select PostgreSQL. Fill in with the host name from that you copied in the previous step and the rest of the parameters (database name, password). Then test the connection.  |  |
| 3  | Change database URL and dependencies in the project’s files  | Change the docker-compose.yml file y database.py of the project, filling in with the corresponding username:password@host (with no database in the end) Remove from the docker-compose.yml of our project the postgres service and the ”depend_on” of postgres from the rest of the services (because it will cease to exist, being replaced by the RDS instance)  |  |
| 4  | Test app  | Run docker compose up on your EC2 instance. Go to host:9000 and fill in the form. Check changes locally using DBeaver.  |  |


