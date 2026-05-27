# Table of Contents

- [Table of Contents](#table-of-contents)
    - [CICD Pipeline architecture using jenkins](#cicd-pipeline-architecture-using-jenkins)
        
    - [Configure github webhook with jenkins (CI)](#configure-github-webhook-with-jenkins-ci)
        
        - [Creating webhook](#creating-webhook)
            
        - [Create the SSH key](#Create-the-SSH-key)
            
        - [Adding the public SSH key for github repo access](#Adding-the-public-SSH-key-for-github-repo-access)
            
    - [Merge github branches(CI) using jenkins](#Merge-github-branches(CI)-using-jenkins)
        
        - [Using git publisher plugin for merging dev branch to main (preferred method)](#using-git-publisher-plugin-for-merging-dev-branch-to-main-preferred-method)
    - [Deploying sparta app v1.2 to app running on EC2 using jenkins(CD)](#Deploying-sparta-app-v1.2-to-app-running-on-EC2-using-jenkins(CD))
        
    - [Testing pipeline](#testing-pipeline)
        

* * *

## CICD Pipeline architecture using jenkins


 ![excalidraw.svg](_resources/excalidraw-a07f76bc879f4f5c9ae7f2099c11f367.svg) 

 
Why did we setup the CICD pipeline the way we did, benefits you have seen, benefits for an organisation?

*The CICD pipeline was split up into 3 jobs each run by a seperate agent node. The benefits of seperating the jobs were that the jobs could be run independantly of each other and the modular design means it can be adapted for other projects. Additionally, it means that issues can be more quickly identified if the task is broken down as the blocker will be apparent in the pipeline due to it halting the pipline, inhibiting the next job. A identification of issues is important for buisnesses as it allows problems to be tackled quicker.*

How did you setup each of your jobs (including authentication/security), webhook and how was the pipeline is triggered, what should the results be at the end?

*Authentication was done by giving jenkins access to private SSH keys to access the github repo as well as the EC2 instance running the app. The pipline was triggered an initial push to the dev branch which triggered the webhook, the deployment of the app was triggered by completion of a merge of the dev branch to main. The result at the end should be that edits to the dev branch should be deployed to the app running on the E2 instance.*

## Configure github webhook with jenkins (CI)

1.  Using a web browser navigate to the server via http://52.31.15.176:8080
    
2.  Login
    
3.  Select ---> new item
    
4.  Follow the configuration below;
    
    | Section | Setting |
    | --- | --- |
    | Project Name | burhan-job1-ci-test  <br><br/>freestyle project |
    | Build Retention | Keep max 5 builds |
    | GitHub Project | https://github.com/vrangr-ops/tech603-ttt-app-cicd-jenkins/ |
    | SCM Type | Git |
    | Repository URL | git@github.com:vrangr-ops/tech603-ttt-app-cicd-jenkins.git |
    | Credentials | Global credentials (SSH Username with Private Key)  <br><br/>ID and Username--> use SSH key name  <br><br/>paste private key   <br><br/> |
    | Branches to Build | \*/dev |
    | Build Triggers | GitHub hook trigger for GITScm polling |
    | Build Environment | NodeJS Installation: v20 |
    | Build Steps (Shell) | cd app  <br><br/>npm ci  <br><br/>npm test |
    

- npm ci is a command used in Node.js projects to install dependencies directly from the package-lock.json file, ensuring a clean and consistent installation of packages.
    
    - It is faster and more reliable than npm install, especially in automated environments like continuous integration.
- npm test is used to execute tests defined in your project  
    <br/>
    

**Connecting jenkins to github**  

![e4c90a2f6cf4b4aec513490cf26c3425.png](_resources/e4c90a2f6cf4b4aec513490cf26c3425.png)


### Creating webhook

1.  Navigate to github repository
2.  Settings
3.  Webhooks
4.  Add webhooks
5.  Paste in the URL of the jenkins server as the playload url with the webook extension
    - http://52.31.15.176:8080/github-webhook/
6.  Event trigger --> **just the push event**
7.  Save the webhook

&nbsp;

### Create the SSH key

1.  Using bash terminal navigate to `~/.ssh`
2.  Generate key RSA key

- `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`

3.  Initialise the agent and set up environment variable

- `eval "$(ssh-agent -s)"`

4.  Add the SSH private key to the ssh-agent

- `ssh-add ~/.ssh/github`

### Adding the public SSH key for github repo access

1.  Navigate to the repo
2.  Select --> settings
3.  Add deploy key
4.  Use the name of the SSH key --> `name-XXX-XXX-key.pub`
5.  Paste the public key
6.  Save

&nbsp;

## Merge github branches(CI) using jenkins

1.  Using a web browser navigate to the server via http://52.31.15.176:8080
2.  Login
3.  Select ---> **new item**
4.  Follow the configuration below;

| Section | Setting |
| --- | --- |
| Name | burhan-job2-ci-merge  <br><br/>freestyle project |
| Discard old builds | Max # of builds to keep : 5 |
| GitHub project | https://github.com/vrangr-ops/tech603-ttt-app-cicd-jenkins/ |
| Source Code Management   <br><br/>Repository URL  <br><br/>Add --> jenkins  <br><br/>Branches to build | Git  <br><br/>git@github.com:vrangr-ops/tech603-ttt-app-cicd-jenkins.git  <br><br/>Domain: Global credentials  <br><br/>SSH username with Private key---> paste key  <br><br/>\*/dev |
| Build Triggers | Build after other projects are built |
| Provide Node & npm bin/ folder to PATH | NodeJS Installation:v20  <br><br/>SSH agent  <br><br/>   <br><br/>   <br><br/> |
| SSH Agent | tech603-burhan-public-github-jenkins   <br><br/> |
| Build Steps | Execute shell;  <br><br/><br/>git fetch origin  <br>git checkout main  <br>git pull origin main  <br>git merge origin/dev --no-edit  <br>git push origin main |

**Successful build**  

![49877f4f3a232754be9c8921903bbb15.png](_resources/49877f4f3a232754be9c8921903bbb15.png)  

<br/>

### Using git publisher plugin for merging dev branch to main (preferred method)

1.  Using a web browser navigate to the server via http://52.31.15.176:8080
    
2.  Login
    
3.  Select ---> new item
    
4.  Follow the configuration below;
    

| Section | Setting |
| --- | --- |
| Name | burhan-job2-ci-merge-plugin  <br><br/>freestyle project |
| Discard old builds | Max # of builds to keep : 5 |
| GitHub project | https://github.com/vrangr-ops/tech603-ttt-app-cicd-jenkins/ |
| Source Code Management   <br><br/>Repository URL  <br><br/>Add --> jenkins  <br><br/>Branches to build | Git  <br><br/>git@github.com:vrangr-ops/tech603-ttt-app-cicd-jenkins.git  <br><br/>Domain: Global credentials  <br><br/>SSH username with Private key---> paste key  <br><br/>\*/dev |
| Build Triggers | Build after other projects are built |
| Provide Node & npm bin/ folder to PATH | NodeJS Installation:v20  <br><br/>SSH agent |
| SSH Agent | tech603-burhan-public-github-jenkins |
| Post-build Actions | **Git Publisher;**[](http://52.31.15.176:8080/job/burhan-job2-ci-merge-plugin/configure "Help")  <br><br/>tick --> Push Only If Build Succeeds  <br><br/>tick --> Merge Results  <br><br/>Branch to push --> main  <br><br/>Target remote name --> origin |

&nbsp;

**Successful build**

**![7890c817db3a57f08fc87101d2c1d168.png](_resources/7890c817db3a57f08fc87101d2c1d168.png)**


## Deploying sparta app v1.2 to app running on EC2 using jenkins(CD)

1.  Using a web browser navigate to the server via http://52.31.15.176:8080
2.  Login
3.  Select ---> **new item**
4.  Follow the configuration below;

| Section | Setting |
| --- | --- |
| Name | burhan-job3-cd-deploy  <br><br/>freestyle project |
| Discard old builds | Max # of builds to keep : 5 |
| GitHub project | https://github.com/vrangr-ops/tech603-ttt-app-cicd-jenkins/ |
| Source Code Management | Git |
| Repository URL | git@github.com:vrangr-ops/tech603-ttt-app-cicd-jenkins.git |
| Add --> jenkins | Domain: Global credentials   <br><br/>SSH username with Private key---> select key |
| Branches to build | \*/dev |
| Build Triggers | Build after other projects are built |
| Provide Node & npm bin/ folder to PATH | NodeJS Installation:v20 |
| SSH Agent | add ---> tech503-burhan-aws.pem  <br><br/>tech603-burhan-public-github-jenkins  <br><br/>paste --> private key  <br><br/> |
| Build Steps | Execute shell;[](http://52.31.15.176:8080/job/burhan-job1-ci-test/configure "Help")  <br><br/>\# 1. Copy the files  <br>scp -o StrictHostKeyChecking=no -r app ubuntu@54.170.163.72:/home/ubuntu/repo/tech603-sparta-app/nodejs20-sparta-tictactoe-v1/  <br><br/>\# 2. ssh in  <br>ssh -o StrictHostKeyChecking=no ubuntu@54.170.163.72 << 'EOF'  <br>  # Load environment to find node/npm/pm2  <br>  \[ -f ~/.bashrc \] && . ~/.bashrc  <br><br/>  echo "Connected and starting remote tasks..."  <br><br/>  # Navigate to app  <br>  cd /home/ubuntu/repo/tech603-sparta-app/nodejs20-sparta-tictactoe-v1/app  <br><br/>  # start the app  <br>  pm2 kill  <br>  npm install  <br>  pm2 start index.js --name "sparta app"  <br><br/>  echo "Deployment finished successfully."  <br>EOF |

**Successful build**


![7ace4a16ad12f8bd0fb279a0ee36c231.png](_resources/7ace4a16ad12f8bd0fb279a0ee36c231.png)


&nbsp;

## Testing pipeline

Original timestamp  

![b1cc500e0b71f8f0e0a301db16a12a7f.png](_resources/b1cc500e0b71f8f0e0a301db16a12a7f.png)


1.  Navigate to the app folder using a bash terminal
2.  `git status`
3.  `git checkout origin dev`
4.  Edit the `server.js` file using text editor ---> save the file  

    ![3f02ae56cea853598bc0f5b1ae35ac63.png](_resources/3f02ae56cea853598bc0f5b1ae35ac63.png)

5.  `git add .`
6.  `git commit -m "timestamp change"`
7.  `git push origin dev`
8.  Navigate to the app instance by pasting the public IP using http from a web browser


![da3403040c7e427409ae32868bb93fd3.png](_resources/da3403040c7e427409ae32868bb93fd3.png)

