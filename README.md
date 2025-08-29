# Python App Deployment using jenkins pipeline

> This project helps you how to deploy a Python  application on a **Remote server** using jenkins pipeline .
for Continuous integration and continuous deployment .

## Objective :   

* #### Automate Build , Test , and Deployment of a Python application  .

* #### Implement a CICD pipeline using Jenkins Declarative pipeline  . 

* #### Deploy the application on a remote server  .
 
 ---

## Prerequisites :

1. ### Infrastructure 

   * **Jenkins Server** (to run the pipeline)

   * **Remote Server** (to host the application)

2. ### Software Requirements 

   * Remote Server with :

     * Python 3.x installed 

     * Pip installed 

   * Jenkins server with :
     * Jenkins installed 

     * Java installed 

     * Plugins installed 

     * Github repository with your node.js application 

   * Plugins Used : 
     * [Github plugin](https://plugins.jenkins.io/github/)
   
     * [SSh Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

     * [Pipeline Plugin](https://plugins.jenkins.io/workflow-aggregator/)

3. ### Security Groups 

    * Allow Port :
      * 22 ( For SSH Access)
      
      * 8080 (For Jenkins)

      * 5000 ( For Python)
   
* Both server launch using the same key 


## Setting Up Jenkins Credentials 

### Step 1 : Copy your pem key on jenkins server 

#### From your local machine : 
bash 

      scp -i pem-linux-key.pem pem-linux-key.pem ubuntu@<jenkins-server-public-ip>:/home/ubuntu


### Step 2 : Add Pem Key To Jenkins Credentials 

1. Go to settings --> credentials --> Under stored scoped to jenkins -> Click (global) --> Add credentials 

2. #### Fill In : 
   *  **Kind** : SSH Username with private key 
   
   *  **ID**   : `python-app-key-credentials` 
   
   *  **Username** : `Ubuntu`
   
   *  **private key** : Choose **Enter directly** and paste the contents of `key which store on jenkins`

![my image](./images/Screenshot%202025-08-30%20031942.png)

3. #### Click Create :

![my image](./images/Screenshot%202025-08-30%20031531.png)


## Create jenkins Pipeline Job

1. **Create Job** : Python-app-deployment 

2. **Job type**  : Pipeline 

3. Under **Pipeline script from scm** : 
      * Choose : Git 

      * Connect to your github repo 

![my image](./images/Screenshot%202025-08-30%20032217.png)

4. Add a jenkinsfile in your repo 
---
### Jenkinsfile for Remote Deployment

  bash


    pipeline {
       agent any

    environment {
        SSH_CRED = 'SSH-credentials-for-python-app'
        SERVER_IP = 'python-app-server-ip'
        REMOTE_USER = 'ubuntu'
        APP_DIR = '/home/ubuntu/pythonapp'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/Nikhil-patil678/Python-App-CICD.git', branch: 'main'
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(credentials: ["${SSH_CRED}"]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} "mkdir -p ${APP_DIR}"
                        scp -r Dockerfile README.md app.py requirements.txt test ${REMOTE_USER}@${SERVER_IP}:${APP_DIR}/
                    '''
                }
            }
        }

        stage('Install & Run App') {
            steps {
                sshagent(["${SSH_CRED}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            sudo apt update &&
                            sudo apt install -y python3-venv python3-pip &&
                            cd ${APP_DIR} &&
                            python3 -m venv venv &&
                            source venv/bin/activate &&
                            pip install --upgrade pip &&
                            pip install -r requirements.txt &&
                            nohup python3 app.py --host=0.0.0.0 > app.log 2>&1 &
                            exit 0
                        '
                    """
                }
            }
        }
      }
    }



---

5. Build the job :

#### Console Output Showing Succeesful Deployment 

![my image](./images/Screenshot%202025-08-30%20032956.png)

* ### Access URL : 

bash 

    http://<public-ip of app-server>:5000


## Setup Github Hooks 
 
 ### Step 1 : Enable Github hook Trigger In Jenkins Job 
 1. Open your Jenkins job --> Configure 

2. Under Triggers ckeck : 
    * `GitHub hook trigger for GITScm polling`

![my image](./images/Screenshot%202025-08-30%20034346.png)

### Step 2 : Add Webhook In Github Repo 
1. Go to github repo -->  **settings** -> **Webhooks** --> **Add Webhooks** --> 

2. Under Payload URL Enter  : 

   *  `http://< jenkins-server-ip >:8080/github-webhook/`

- Content Type : application/x-www-form-urlencoded 

- Trigger : Just the push event

![my image](./images/Screenshot%202025-08-30%20034927.png)

## Summary : 

#### This project demonstrates end-to-end CI/CD deployment of a python application using jenkins pipelie .

The pipeline automatically : 

1. Clone the source code from github .

2. Install dependencies using pip and requirement.txt .

3. Runs unit test eith `pytest` .

4. Deploy the application on a remote server via SSH . 

## Author : 

**NIKHIL PATIL**

📧 np3250072@gmail.com

🔗 **[Linkdein]** : https://www.linkedin.com/in/nikhil-patil-259132306/ 

🔗 **[Github]**  : https://github.com/Nikhil-patil678 

---









         