# Github-Jenkins-Docker_IntegrationProject
Automation is future to our world, Here are my steps towards automation made a small project based on Integration of Github over Jenkins using docker image httpd for automation purpose. This project covers the day-to-day problems faced by the Developer, Product Manager, and Quality Assurance team so to overcome those problems here is the small demo for achieving automation.
# Tasks:
For this project for automation process, We have to make 3 jobs on Jenkins:
### JOB#1
If Developer push to dev1 branch then Jenkins will fetch from dev and deploy on the dev-docker environment.
### JOB#2
If Developer push to master branch then Jenkins will fetch from master and deploy on the master-docker environment. As they both dev-docker and master-docker environments are on different docker containers.
### JOB#3
Manually the QA team will check (test) for the website running in the dev-docker environment. If it is running fine then Jenkins will merge the dev branch to master branch and trigger #job 2
## Built with
- RedHat Linux RHEL8
- Docker
- Git Bash
- Jenkins
# Pre-requisite :
- OS: Base OS is Windows 10. Server OS is RedHat Enterprise Linux 8 (RHEL8) in Virtual Box.
- In RHEL8 some of the software needed is Docker (also need the httpd image downloaded in it), Jenkins (also GitHub plugin should be installed in it), ngrok program.
```
docker pull httpd
unzip ngrok.tar.gz
```
- In Windows, we have to install git bash software.
- At the start stop the firewall in RHEL8 and start the docker and Jenkins services.
```
systemctl stop firewalld
systemctl disable firewalld
yum install jenkins
yum install java
yum install httpd
```
- For configuration of `yum.repos.d` and more, Refer my blog her: https://www.linkedin.com/pulse/iiecrise-docker-project-rohit-ghumare
# Step-by-Step Approach towards automation world:
### 1. Giving Jenkins all the power of Linux :
For running Jenkins with RHEL8, You can face some permission errors like for credentials or writing or reading files that's why to edit the sudoers file using: `gedit /etc/sudoers` in your RHEL8. Follow the below image and add the highlighted lines i.e `jenkins  (ALL:ALL) NOPASSWD:ALL`.
![sudoer](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/sudoers.PNG)
### 2. Necessary git commands to run inside git bash app:
- Make directory to clone your repository and afollow the steps given in below images:
```
mkdir Automation
cd Automation/
git clone "https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/"
```
- Creating master branch and commit changes over the same.
![git](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/git%20bash%20cmd.PNG)
- Checkout from master and commit from developer area/zone.
![git0](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/git%20bash%20cmd1.PNG)
- Git commands for commits, push and switching
![git1](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/git%20bash%20cmd2.PNG)
![gitcommit](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/git%20commit.PNG)
- If sometimes, Developer is ahead by commits in comparison with devloper than simply run following commands:
```
git checkout master
git log --online
git merge dev1
git branch dev1
git rebase
git log --online --graph --decorate
```
- For automation, Edit post-commit inside `.git/hooks` and use `Esc, :wq` to get out of vim:
```
vi post-commit
#!bin/bash
git push
```
#### 3. Creating the developer Job in Jenkins :
Follow the below-mentioned image and start to build your first job on Jenkins. Note that, This job is for the Developer Unit and hence this Job will pull the data from the master branch and store it, Jenkins will fetch devloper and deploy the server using docker by suing shell commnads automatically.
- Setting up the git repository for kaam1 on our jenkins dashboard by using config jenkin item, Enter Repository URL for your github repo inside SCM as shown below:
![kaam1](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam1.1.PNG)
- configure **Build Triggers** by using POLL SCM and select 5 stars for running environment by each minutes each star depicts time keywords:
![kaam10](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam1.2.PNG)
- This is the main job inside Jenkins to make environment to work automatically, Edit below code inside **Execute Shell** to configure the docker environment inside RHEL8 it shows if erver is running than it will print Production server already running otherwise it will create new server using httpd docker container:
```
if sudo docker ps | grep production_server
then
echo "Production server already running"
else
sudo docker run -d -t -i -p 8081:80 -v /production:/usr/local/apache2/htdocs/ --name production_server httpd
fi
sudo cp -r -v -f * /production/
```
![kaaam100](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam1.3.PNG)
#### 4. Creating the Tester Job in Jenkins :
Follow the below-mentioned image and start to build your second job on Jenkins. Note that, This job is for the Tester Unit and If Developer push to master branch then Jenkins will fetch from master and deploy on the master-docker environment. Hence, Tester will test the dev1 and master will make necessary deployment by running new tester server.
- Setting up the git repository for kaam2 on our jenkins dashboard by using config jenkin item, Enter Repository URL for your github repo inside SCM as shown below:
![kaam2](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam2.PNG)
- configure **Build Triggers** by using POLL SCM and select 5 stars for running environment by each minutes each star depicts time keywords:
![kaam20](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam21.PNG)
- This is the main job inside Jenkins to make environment to work automatically, Edit below code inside **Execute Shell** to configure the docker environment inside RHEL8 it shows if erver is running than it will print Production server already running otherwise it will create new server using httpd docker container:
```
if sudo docker ps | grep tester_server
then
echo "Tester server already running"
else
sudo docker run -d -t -i -p 8082:80 -v /tester:/usr/local/apache2/htdocs/ --name tester_server httpd
fi
sudo cp -r -v -f * /tester/
```
![kaaam200](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam22.PNG)
### 5. Creating the Quality Assurance Job in Jenkins to issue Certificate:
- This job is the most important and here is the actual magic begins, It merge the developer and master branches by running necessary shell commands automatically, It internally runs some build projects and triggers to merge the developer and tester server.
- Setting up the git repository for kaam2 on our jenkins dashboard by using config jenkin item, Enter Repository URL for your github repo inside SCM and add username and password for credentials ad also **Add merge before build** as shown below:
![kaam3](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam3.PNG)
- Build triggers : No need to add this trigger , as testing team will run this job but assuming that all tests are good so we trigger this job just after kaam2:
![kaam30](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam3.2.PNG)
- Add Build other projects and Git publisher as shown below:
![kaam300](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/kaam3.3.PNG)
### 6.After successfull completion of above commands you can easily see the results as given below:
![mpg](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/jenkin%20main%20pg.PNG)
![dev](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/developer.PNG)
![tester](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/tester.PNG)
![bash](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/gith%20bash.PNG)
![httpd](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/httpd%20server.PNG)
# Tunelling for webserver:
- Tester can check the website without having IP of the RHEL, Simply do tunelling for the same.
- Use ngrok.io for tunelling
```
./ngrok http 8081
```
# IDEA
- Proposing idea for Covid19 situation in india, We can deploy, create and do webserver configuration between developer and Tester to get full flegged website just with one-click.
- In one click, docker will create containers and also configure github and jenkins and integrate over docker.
- Check out this demo website, We can create same for Red zone, Green zone and orange zone in india.
![imgmap](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/Images/map.PNG)
# Check this public website: http://ffe57dc7.ngrok.io/mymap.html
# Author
[**Rohit Ghumare**](https://github.com/rohitg00)
## License
[**Apache License**](https://github.com/rohitg00/Github-Jenkins-Docker_IntegrationProject/blob/master/LICENSE)
