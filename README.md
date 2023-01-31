# Setting up a CI/CD Pipeline for a node.js app with jenskin

## This project is an example of how to setup a simple CI/CD pipeline for a node.js app with jenkins. Every part of this project is simple code and commands which shows how to;

* summary of what CI/CD is
* creating simple Node App
* Writing test
* Serve and configure Node App Server
* Set up and configure Jenkins Server
* Deployment of node app

## What is CI/CD?

Continuous Integration and Continuous Deployment are two modern software development practices. Continuous Integration (CI) is the process of automating the build and testing of code every time a team member commits changes to version control. Continuous Deployment (CD) can be thought of as an extension of continuous integration, and is the process of automatically deploying an application after CI is successful. The goal is to minimize lead time; the time elapsed between development writing one new line of code and this new code being used by live users in production.

## Why CI/CD?

There many benefits for CI/CD practices. I am not going to talk about each benefit in detail but I would like to highlight few of them here:

### Continuous Integration Benefits:

Fewer bugs
Less context switching as developers are alerted as soon as they break the build
Testing costs are reduced
Your QA team spend less time testing.

### Continuous Deployment Benefits:

Releases are less risky
Easy release
Customers see a continuous stream of improvements
Expedite development as there’s no need to pause development for releases

## Creating a simple Node App

Before we write any CI/CD pipeline we need an application to test and deploy. We are going to build a simple node.js application that responds with “hello world” text. 

copy the index.js, package.json from my repo above for sample code.

Run 
"" npm install ""
"node index.js"

You can view the app on your local browser on http://localhost:3000

Now clone your repo down to your local machine using git clone command

## Serve Node App on Amazon Linux ec2-instance

create a free tier linux based ec2-instance on amazon, name it node-server and ssh into it from your favorite command line.

"ssh -i keypair ec2-user@node-server-ip"

switch to root user using the command 

"sudo su - "

create user we don't want to use our root user for security reasons

"adduser <username>
"passwd <new password>

Give the new user sudo priveleges using the command

"usermod -aG sudo <username>

now switch to the new user 

"su - username"

install node and npm on this server by refering to node.js official website

install git 

"sudo yum install git"

navigate to project folder and install all dependencies 

"cd node-app"
"npm install"

make sure to allow access from port 3000 from you node-server ec2-instance security group.

now run 

"node index.js" 

access your node app on http://node-server.ip:3000

#### Running Node App forvever

Starting node app like above is good for development purposes but not in production. In case our node instance crash we need a process that will do the auto restart. We are going to use PM2 module to help us with this task. PM2 is a general purpose process manager and a production runtime for Node.js apps with a built-in Load Balancer. Let’s install PM2 and start our node instance:

"sudo yum install pm2@latest -g
"pm2 start index.js"

now our node server is configured and running our our app.

## Set up Jenkins server

### creating jenkins server

create a second ec2-instance with name jenskins-server and follow same steps as we did for the node-server up untill git install

#### Install Jenkins

get jenkins 

"wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add - "
"sudo yum install jenkins"

start jenkins 

"sudo systemctl status jenskins"

jenskins run on port 8080. remember to allow port 8080 open on ec2-security group.

and now jenkins can be access on http:jenkins-server.ip:8080

### configure jenkins 

When you navigate to Jenkins homepage you probably noticed additional step you need to do. You need to unlock Jenkins

Copy the Jenkins password hosted on your Jenkins server

"sudo cat <path mentioned on the unlock page"

Paste the the password into the text field. You are ready to set up Jenkins. choose the install recommended plugins. Or From the left menu select manage Jenkins and go to manage plugins. On the plugins page select the available tab and look for GitHub plugin, select its checkbox and click the Download now and install after restart button. and restart after installation is complete

### change your jenkins admin password

I suggest at this point to change your Jenkins admin user password. Select Manage Jenkins from the left menu and click on Manage Users. Select the admin user and choose a new password. You will use the new password when you log in to Jenkins in the future.

### create jenkins job

We are going to create our Jenkins job that will be responsible for pulling code changes from node-app git repo, install dependencies and deploy the application every time a developer push changes to the nodejs-app repo main branch.

Click on New Item button, name the item node-app and select Build a free-style software project option and click the OK button.

### configure jenkins job

Source Code Management: Select git radio button and enter github https link to the node-app repo:

"https://github.com/<username>/node-app.git"

Build Triggers: select option GitHub hook trigger for GITScm polling. This will start our Jenkins job on every git push on the main branch. 

click apply.

### Add Git Webhook

We are going to add Git Webhook to inform Jenkins every time a developer push new code to master branch.
Go to node-app GitHub, click on the Settings tab, select Webhooks from the left menu and click on the Add Webhooks button. Enter your Jenkins webhook URL under Payload URL:

" http://JENKINS.SERVER.IP:8080/github-webhook/ "
and select Just the Push Event option. click the add webhook button.

Let’s test what we have so far. Go to your node-app project on your machine and change the version in the package.json to 0.0.2. Commit and push this change to GitHub. After you push, go to your Jenkins job on the browser and observe that the Jenkins job started and completed successfully.

## Deployment 

The last piece of the puzzle is deploying our node app application into the node-app server when our code is push.

### ssh Authentication

In order to do that, Jenkins Server will need to ssh into the node-app server, clone the repo, install dependencies and restart the server. Lets set up ssh access to Jenkins first.

SSH into our  jenkins-server and switch to the user created above with sudo privileges.

generate SSH key: 

"ssh-keygen -t rsa"

and save the generated key in your preferred folder ending in .../.ssh/id_rsa

ssh into the node-server and run 

"ifconfig"

copy the private ip adress of the server. switch back to jenkins server and run 

ssh-copy-id <private node server ip>

follow on screen instructions to establish ssh connections between both servers.




