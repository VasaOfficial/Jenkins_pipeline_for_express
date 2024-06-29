1. make an ec2 instance

2. ssh into it

3. update and install required packages
sudo yum update -y
sudo yum install -y yum-utils

4. add terraform repo
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo

5. install terraform
sudo yum -y install terraform

6. verify terraform installation
terraform -v

7. change the hostname
sudo hostnamectl set-hostname Terraform-Server

8. restart the server

9. make s3 bucket

10. make directory for jenkins
mkdir jenkins && cd jenkins


### Nodejs step
in the jenkins server:

1. Switch to the root user
sudo su -

2. Navigate to the /opt directory:
cd /opt

3. Download Node.js
wget https://nodejs.org/dist/v16.15.0/node-v16.15.0-linux-x64.tar.xz

4. Extract the Node.js tarball
tar -xf node-v16.15.0-linux-x64.tar.xz

5. Rename the extracted folder for convenience:
mv node-v16.15.0-linux-x64 nodejs
rm node-v16.15.0-linux-x64.tar.xz

7. Navigate to the Node.js binary directory and verify the installation:
cd nodejs/bin
./node -v
./npm -v

8. Open .bash_profile to edit
vim .bash_profile

9. Add the following lines to set the environment variables for Node.js:
# Add Node.js binaries to PATH
NODE_HOME=/opt/nodejs
PATH=$PATH:$HOME/bin:$NODE_HOME/bin
export PATH

10. Apply the changes:
source .bash_profile

11. Verify the updated PATH and Node.js installation:
echo $PATH
node -v
npm -v
