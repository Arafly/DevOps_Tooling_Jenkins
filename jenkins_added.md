## Tooling Website deployment automation with Continuous Integration.

In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the mostl popular CI/CD tools.

## Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server. Configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how your update architecture will look like upon competion of this project:

* Archi image

## Step 1 - Install Jenkins server
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins”
2. Install JDK (since Jenkins is a Java-based application)


```

done.
araflyayinde@jenkins:~$ java -version
openjdk version "11.0.11" 2021-04-20
OpenJDK Runtime Environment (build 11.0.11+9-Ubuntu-0ubuntu2.20.04)
OpenJDK 64-Bit Server VM (build 11.0.11+9-Ubuntu-0ubuntu2.20.04, mixed 
mode, sharing)
araflyayinde@jenkins:~$ sudo update-alternatives --config java
There is only one alternative in link group java (providing /usr/bin/ja
va): /usr/lib/jvm/java-11-openjdk-amd64/bin/java
Nothing to configure.
araflyayinde@jenkins:~$ sudo nano /etc/environment
araflyayinde@jenkins:~$ source /etc/environment
araflyayinde@jenkins:~$ echo $JAVA_HOME

```
#Installing Jekins

```
araflyayinde@jenkins:~$ sudo systemctl status jenkins
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
May 05 15:58:19 jenkins systemd[1]: Started LSB: Start Jenkins at boot ti>
araflyayinde@jenkins:~$ 
```

Make sure Jenkins is up and running

### Opening the Firewall

sudo systemctl status jenkins
By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group

## Configure Jenkins to retrieve source codes from GitHub using Webhooks

- Enable webhooks in your GitHub repository settings
*image webhook

- Go to Jenkins web console, click “New Item” and create a “Freestyle project”

- To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

- In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.


- Save the configuration and let us try to run the build. For now we can only do it manually. Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under #1

![](https://github.com/Arafly/DevOps_Tooling_Jenkins/blob/master/assets/build.PNG)

- But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

- Click “Configure” your job/project and add these two configurations
Configure triggering the job from GitHub webhook:

- Configure “Post-build Actions” to archive all the files - files resulted from a build are called “artifacts”

![](https://github.com/Arafly/DevOps_Tooling_Jenkins/blob/master/assets/Build%20process.PNG)


Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results - artifacts, saved on Jenkins server.



You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.