# Tooling Website deployment automation with Continuous Integration.

In this project we are going to start automating part of our routine tasks with an open source automation server - Jenkins. It is one of the mostl popular CI/CD tools.

## Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server. Configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how your update architecture will look like upon competion of this project:

* Archi image

## Install Jenkins server
1. Create an  Ubuntu Server 20.04 LTS on any Cloud Provider and name it *Jenkins*.

2. Install JDK (since Jenkins is a Java-based application).
The default Ubuntu 20.04 repositories include two OpenJDK packages, Java Runtime Environment (JRE) and Java Development Kit (JDK). The JRE consists of the Java virtual machine (JVM), classes, and binaries that allow you to run Java programs. The JDK includes the JRE and development/debugging tools and libraries necessary to build Java applications.

### Installing OpenJDK 11

```
sudo apt update
sudo apt install openjdk-11-jdk
```

Verify the installation by checking the Java version:

`$ java -version`

```
Output

openjdk version "11.0.11" 2021-04-20
OpenJDK Runtime Environment (build 11.0.11+9-Ubuntu-0ubuntu2.20.04)
OpenJDK 64-Bit Server VM (build 11.0.11+9-Ubuntu-0ubuntu2.20.04, mixed 
mode, sharing)
```

> We could also have installed the minimal Java runtime - openjdk-11-jdk-headless package
, but because we'd be leveraging Jenkins on this Jave, it's only right we install the full package

### JAVA_HOME Environment Variable
The JAVA_HOME environment variable is used by some Java applications to determine the Java installation location e.g Jenkins.
To set the JAVA_HOME variable, first find the Java installation path with update-alternatives:

`$ sudo update-alternatives --config java`

```
Output:

There is only one alternative in link group java (providing /usr/bin/ja
va): /usr/lib/jvm/java-11-openjdk-amd64/bin/java
Nothing to configure.
```

Once you found the path of your preferred Java installation, open the /etc/environment file:

`sudo nano /etc/environment`

`JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"
`

For changes to take effect on your current shell you can either log out and log in or run the following source command:

`source /etc/environment`

Verify that the JAVA_HOME environment variable was correctly set:

`echo $JAVA_HOME`

```
Output:

/usr/lib/jvm/java-11-openjdk-amd64
```

```
araflyayinde@jenkins:~$ sudo nano /etc/environment
araflyayinde@jenkins:~$ source /etc/environment
araflyayinde@jenkins:~$ echo $JAVA_HOME
```

## Installing Jekins

The version of Jenkins included with the default Ubuntu packages is often behind the latest available version from the project itself. To ensure you have the latest fixes and features, use the project-maintained packages to install Jenkins.

First, add the repository key to the system:

`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
`

After the key is added the system will return with OK.

Next, let’s append the Debian package repository address to the server’s sources.list:

`sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
`

Now we run:

```
sudo apt update
sudo apt install jenkins
```

- Start Jenkins and enable it across reboot.

```
sudo systemctl start jenkins

sudo systemctl enable jenkins
```

Ensure the Jenkins service is running:

`sudo systemctl status jenkins`

```
Output:

● jenkins.service - LSB: Start Jenkins at boot time
     Loaded: loaded (/etc/init.d/jenkins; generated)
     Active: active (exited) since Wed 2021-05-05 15:58:19 UTC; 46s ago
       Docs: man:systemd-sysv-generator(8)
      Tasks: 0 (limit: 4713)
     Memory: 0B
     CGroup: /system.slice/jenkins.service
May 05 15:58:18 jenkins systemd[1]: Starting LSB: Start Jenkins at boot t>
May 05 15:58:18 jenkins jenkins[5777]: Correct java version found
May 05 15:58:18 jenkins jenkins[5777]:  * Starting Jenkins Automation Ser>
May 05 15:58:18 jenkins su[5832]: (to jenkins) root on none
May 05 15:58:18 jenkins su[5832]: pam_unix(su-l:session): session opened >
May 05 15:58:18 jenkins su[5832]: pam_unix(su-l:session): session closed >
May 05 15:58:19 jenkins jenkins[5777]:    ...done.
May 05 15:58:19 jenkins systemd[1]: Started LSB: Start Jenkins...
```

>By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your Security Group or Firewall Rule

### Setting Up Jenkins

To set up your installation, visit Jenkins on its default port, 8080, using your server domain name or IP address: http://your_server_ip_or_domain:8080

You should receive the Unlock Jenkins screen, which displays the location of the initial password:

![](https://github.com/Arafly/DevOps_Tooling_Jenkins/blob/master/assets/jenkins_start.PNG)

Run this command in the terminal to display the password:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Copy the 32-character alphanumeric password from the terminal and paste it into the Administrator password field, then click Continue.

The next screen presents the option of installing suggested plugins or selecting specific plugins:

![](https://github.com/Arafly/DevOps_Tooling_Jenkins/blob/master/assets/jenkins_customize.PNG)

We’ll click the **Install suggested plugins** option, which will immediately begin the installation process.

![](https://github.com/Arafly/DevOps_Tooling_Jenkins/blob/master/assets/plugins.PNG)

When the installation is complete, you’ll be prompted to set up the first administrative user. It’s possible to skip this step and continue as admin using the initial password we used above, but we’ll take a moment to create the user.

After confirming the appropriate information, click Save and Finish. You’ll receive a confirmation page confirming that “Jenkins is Ready!”:

![](https://github.com/Arafly/DevOps_Tooling_Jenkins/blob/master/assets/ready.PNG)

## Configure Jenkins to retrieve source codes from GitHub using Webhooks

This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

- Enable webhooks in your GitHub repository settings
*mage webhook_gif

- Go to Jenkins web console, click “New Item” and create a “Freestyle project”

- To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

- In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

> Add the credentials using the "Jenkins Credentials" and select it when you're done.

- Save the configuration and let us try to run the build. For now we can only do it manually. Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under #1

![](https://github.com/Arafly/DevOps_Tooling_Jenkins/blob/master/assets/build.PNG)

- But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

- Click “Configure” your job/project and add these two configurations
Configure triggering the job from GitHub webhook:

- Configure “Post-build Actions” to archive all the files - files resulted from a build are called “artifacts”

![](https://github.com/Arafly/DevOps_Tooling_Jenkins/blob/master/assets/Build%20process.PNG)


Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results - artifacts, saved on Jenkins server.

*image guit_build

> By default, the artifacts are stored on Jenkins server locally

`$ ls /var/lib/jenkins/jobs/tooling_github/builds/2/a
rchive/`

```
Output:

Dockerfile   README.md           html          tooling-db.sql
Jenkinsfile  apache-config.conf  start-apache
```

##  Configure Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.