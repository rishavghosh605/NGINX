Learning different things on AWS EC2 instance: ubuntu18

1> Yarn

On Debian or Ubuntu Linux, you can install Yarn via our Debian package repository. You will first need to configure the repository:

curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

Note:
On Ubuntu 16.04 or below and Debian Stable, you will also need to configure the NodeSource repository to get a new enough version of Node.js.

Then you can simply:

sudo apt update && sudo apt install yarn

2) Making a seperate user when running your applications

adduser user_name //Then you can proceed on to give your details and your password

usermod -aG wheel rishav //If you want to give rishav admin rights then we need to add it to the wheel group

sudo su - rishav //To become the user rishav

whoami //A command to check which user you are.

mkdir .ssh //We create a ssh folder to connect directly with the non root user later
chmod 700 .ssh //So only the current user in this case only rishav will be able to see it
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys //We make permission to this file even more restricitve

cat ~/.ssh/id_rsa.pub  //To use the public key of "your" local machine

vim .ssh/authorzed_keys //Then add the public key to authorized keys by using vim

tar czf easyio-master.tar.gz main.js package.json yarn.lock public LICENSE  //We create a zip file to be uploaded to the ec2 instance

scp .\easyio-master.tar.gz rishav@172.31.10.160:~ //We can upload it from our command prompt using scp but we probably need to add the private key that we had recieved earlier

In the EC2 instance:
tar xf easy-master.tar -C easyio// To copy paste teh contents of the given zip file into the folder called easyio

yarn install//To download and run the dependencies

**Important
When we use node js in sessions like putty and we use node app.js or npm start like commands they only last as long as the session we have created. If we were to log off from the session then the node js application will close too.

To solve this problem we use pm2

Note: To use scripts that run applications when the server starts we must be root user

pm2 startup systemd -u rishav --hp /home/rishav

sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u rishav --hp /home/ubuntu

