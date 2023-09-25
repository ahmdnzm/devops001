# CI/CD Pipeline - GitHub, Jenkins, Maven, SonarQube, Docker Hub, trivy, Argo CD, Slack
Create a new repository in GitHub name devops001
clone the repository

```bash
git clone https://github.com/ahmdnzm/devops001.git
```

### Launch Jenkins Master instance

Login to AWS Console

go to EC2 dashboard

Click Instances → Instances

Click Launch instances button

On Launch an instance page, fill the following field

Name: Jenkins-Server

AMI choose ****Ubuntu Server 22.04 LTS (HVM), SSD Volume Type****

Instance type: t2.micro

Create new key pair

Key pair name: devops-key

Click Edit on Network settings

Security group name: Jenkins-Server-sg

Description: Jenkins-Server-sg

Inbound:

Type: ssh

Source type: My IP

Description: SSH from My IP

Click `Add security group rule` to add another rule for Jenkins Server Web Access

Type: Custom TCP

Port Range: 8080

Source Type: My IP

Description: Jenkins Server Web Access

Configure storage: 15GiB gp2

Click Launch Instance

Copy key pair pem file to devops001 repository

Change devops-key.pem permission

```bash
chmod 600 devops-key.pem
```

Copy Jenkins-Server public IP address

Ssh to Jenkins-Master instance

```bash
ssh -i devops-key.pem ubuntu@54.161.204.173
```

Once login to Jenkins-Master instance, do system update

```bash
sudo apt update && sudo apt upgrade -y
```

Change hostname to Jenkins-Master

```bash
sudo hostnamectl set-hostname Jenkins-Master
```

Confirm the hostame is changed

```bash
hostnamectl
```

```bash
Static hostname: Jenkins-Master
       Icon name: computer-vm
         Chassis: vm
      Machine ID: fcc18587748c4880a4e8c0480b6a6e48
         Boot ID: 21cd9ae34ea64e0a901a31a97afb8187
  Virtualization: xen
Operating System: Ubuntu 22.04.3 LTS
          Kernel: Linux 5.19.0-1025-aws
    Architecture: x86-64
 Hardware Vendor: Xen
  Hardware Model: HVM domU
```

Reboot server to apply changes

```bash
sudo init 6
```

After a few minutes, login back to Jenkins Master server
Install OpenJDK 17 jre

```bash
sudo apt install openjdk-17-jre -y
```

Confirm java version

```bash
java -version
```

```bash
openjdk version "17.0.8.1" 2023-08-24
OpenJDK Runtime Environment (build 17.0.8.1+1-Ubuntu-0ubuntu122.04)
OpenJDK 64-Bit Server VM (build 17.0.8.1+1-Ubuntu-0ubuntu122.04, mixed mode, sharing)
```

Go to https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

Copy the Weekly Release installation command and paste in command line to start Jenkins installation

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

Once installation done, check Jenkins service is running

```bash
sudo systemctl status jenkins
```
### Launch Jenkins Agent instance

Click Launch instances button

On Launch an instance page, fill the following field

Name: Jenkins-Agent

AMI choose ****Ubuntu Server 22.04 LTS (HVM), SSD Volume Type****

Instance type: t2.micro

Use devops-key key pair

Key pair name: devops-key

Click Edit on Network settings

Security group name: Jenkins-Agent-sg

Description: Jenkins-Agent-sg

Inbound:

Type: ssh

Source type: My IP

Description: SSH from My IP

Configure storage: 15GiB gp2

Click Launch Instance

Once Jenkins-Agent instance created, ssh to the instance

```bash
ssh -i devops-key.pem ubuntu@18.207.196.123
```

Once login to Jenkins-Agent instance, do system update

```bash
sudo apt update && sudo apt upgrade -y
```

Change hostname to Jenkins-Agent

```bash
sudo hostnamectl set-hostname Jenkins-Agent
```

Confirm the hostame is changed

```bash
hostnamectl
```

```bash
Static hostname: Jenkins-Agent
       Icon name: computer-vm
         Chassis: vm
      Machine ID: 6f3a8eaff72d49e28e1c80b0e41005db
         Boot ID: 439f1280ae6e46f0ab71a2c187fc7716
  Virtualization: xen
Operating System: Ubuntu 22.04.3 LTS
          Kernel: Linux 5.19.0-1025-aws
    Architecture: x86-64
 Hardware Vendor: Xen
  Hardware Model: HVM domU
```

Reboot server to apply changes

```bash
sudo init 6
```

After a few minutes, login back to Jenkins Agent instance

Install OpenJDK 17 jre

```bash
sudo apt install openjdk-17-jre -y
```

Confirm java version

```bash
java -version
```

```bash
openjdk version "17.0.8.1" 2023-08-24
OpenJDK Runtime Environment (build 17.0.8.1+1-Ubuntu-0ubuntu122.04)
OpenJDK 64-Bit Server VM (build 17.0.8.1+1-Ubuntu-0ubuntu122.04, mixed mode, sharing)
```

Install docker in Jenkins-Agent instance

```bash
sudo apt install docker.io -y
```

Grant permission to current user to execute docker

Current groups in $USER

```bash
groups $USER
```

```bash
ubuntu : ubuntu adm dialout cdrom floppy sudo audio dip video plugdev netdev lxd
```

Add $USER in docker group

```bash
sudo usermod -aG docker $USER
```

Check $USER in docker group

```bash
groups $USER
```

```bash
ubuntu : ubuntu adm dialout cdrom floppy sudo audio dip video plugdev netdev lxd docker
```

Note: If want to remove docker in $USER’s group

```bash
sudo deluser $USER docker
```

## Edit SSH config file at Jenkins-Master and Jenkins-Agent instance

Edit /etc/ssh/sshd_config

```bash
sudo vim /etc/ssh/sshd_config
```

Uncomment the following line

```bash
PubkeyAuthentication yes

AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
```

Save and exit the editor

Reload sshd service to read modified SSH configuration file

```bash
sudo service sshd reload
```

### SSH Key at Jenkins-Server and Jenkins-Agent

Generate SSH Key at Jenkins-Server and Jenkins-Agent instance. Use default option.

```bash
ssh-keygen -t ed25519
```

At Jenkins-Master instance, view id_ed25519.pub

```bash
cat /home/ubuntu/.ssh/id_ed25519.pub
```

Copy content of id_ed25519.pub into authorized_keys in Jenkins-Agent instance

```bash
vim /home/ubuntu/.ssh/authorized_keys
```

Done copied, try ssh Jenkins-Agent’s private IP from Jenkins-Server

```bash
ssh ubuntu@172.31.91.244
```

### Access Jenkins Web App

To access, Jenkins Web App, use the Jenkins-Master public ip with port 8080

```bash
http://18.204.217.59:8080/
```

To Unlock Jenkins, copy the link into the console to get the password

```bash
sudo cat **/var/lib/jenkins/secrets/initialAdminPassword**
```

Click Install Suggested Plugins

Create User and Start Using Jenkins

### Disable Jenkins-Master Node

On Jenkins Dashboard, click Manage Jenkins → Nodes → Built-In Node → Configure

Change Number of executors to 0 and Save

### Add Jenkins-Agent as new Node

On Jenkins Dashboard, click Manage Jenkins → Nodes → New Node

Node name: Jenkins-Agent

Select Type Permanent Agent

Click Create

Description: Jenkins-Agent

Number of executors: 2

Remote root directory: /home/ubuntu

Labels: Jenkins-Agent

Usage: Use this node as much as possible

Launch Method: Launch agents via SSH

Host: 172.31.91.244 ## Jenkins-Agent private IP

Credentials: Add → Jenkins

In Add Credentials window

Kind: SSH Username with private key

Scope: Global

ID: Jenkins-Agent

Description: Jenkins-Agent

Username: ubuntu

Private Key: Enter directly

*****Copy the content of id_ed25519 in Jenkins-Master instance in Key column

```bash
cat /home/ubuntu/.ssh/id_ed25519
```

******

Credentials: ubuntu (Jenkins-Agent)

Host Key Verification Strategy: Non verifying Verification Strategy

Availability: Keep this agent online as much as possible

Click Save

Check Log at Dashboard → Manage Jenkins → Nodes → Jenkins-Agent for any error

Connection is successful when shown this line in the log

```bash
Agent successfully connected and online
```

### Install Jenkins Plugin

Go to Dashboard → Manage Jenkins → Plugins → Available Plugins

Install the following plugins

- Maven Integration
- Pipeline Maven Integration
- Eclipse Temurin Installer

### Configure Jenkins Plugin

Go to Dashboard → Manage Jenkins → Tools

- Maven Installations
    - Click Add Maven
    - Name: Maven3
    - Use default options
    - Click Apply and Save
- JDK Installations
    - Name: Java17
    - Add Installer: Install from adoptium.net
    - Version: jdk-17.0.5+8
    - Click Apply and Save

### Configure GitHub Integration

Go to Dashboard → Manage Jenkins → Credentials

Under Stores scoped to Jenkins

Click `(global)` dropdown under the Domains. Select Add credentials

![Screenshot 2023-09-24 at 4.59.30 PM.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/50c8a00f-2028-4038-a394-37cc50018e85/643621e1-c33f-4086-a716-6a8eca4147fc/Screenshot_2023-09-24_at_4.59.30_PM.png)

On New Credentials

Kind: Username with password

Scope: Global (Jenkins, nodes, items, all child items, etc)

Username: <GitHub username>

Password: <GitHub Token> #GitHub Settings → Developer Settings → Personal access tokens

ID: github

Description: github

Click Create