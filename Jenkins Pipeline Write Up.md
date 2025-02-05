
==***FYA Every New Service NEEDS its OWN KEYS!!!***==

Generated from # AWS 6.0 - Jenkins Terraform Pipeline 1/29/2025 YouTube Video

A Container is its OWN Machine!! It also needs the Software you are looking to operate on it installed as well.

First I made a new IAM User account just for Jenkins to run through and saved the Access Key and Secret Access Key. These go into the "New Credentials" section further below.

~~this screenshot is just here so I don't lose it.~~
![[Pasted image 20250203153459.png]]

Once Jenkins has been installed, start it up within Docker Desktop. Be sure to Sign in to Docker (Desktop)

Start up Jenkins and select the "Manage Jenkins" option and select "Credentials"
![[Pasted image 20250203151721.png]]
Then click on "Global"
![[Pasted image 20250203151810.png]]
Select "Add Credentials"
![[Pasted image 20250203151856.png]]
While in "New Credentials", select the following options,
"Kind" = AWS Credentials
"ID" = "Description" = can be what you want to name it.
"Access Key ID" = Access key created for new IAM User account in AWS
"Secret Access Key" = Secret Access key created for new IAM User account in AWS
Then select Create.

![[Pasted image 20250203152531.png]]

Then go to "+ New Item"
![[Pasted image 20250203152720.png]]

Once within, give your new pipeline a title and select "Pipeline" from the available options and select "Okay".
![[Pasted image 20250203155025.png]]

Once in the "Configure - General" section, ![[Pasted image 20250203155205.png]]
scroll down to "Advanced Pipeline Options" and select the drop down for "Definition" and select "Pipeline script from SCM" ![[Pasted image 20250203155336.png]]
Then select from the "SCM" dropdown, "GIT"![[Pasted image 20250203155630.png]]
Make sure your "Script Path" matches the file name you are actually using within your Repo for the Jenkins Pipeline.

After that, pull the Repository URL from GitHub you want to use for the Pipeline. 
To make it easy, use a terraform code that you know Already WORKS!!
![[Pasted image 20250203155818.png]]

I used one from another groups project![[Pasted image 20250203160445.png]]
Back in Jenkins, under the "Branches to Build" and in its sub category "Branch Specifier", set it as   "/main", Then select "Save".![[Pasted image 20250203160904.png]]
This is determined by the branch of GitHub you pulled the Repo URL from. ![[Pasted image 20250203161106.png]]
Verify Pipeline creation within the Dashboard.

The Jenkins file is similar to the YAML file in such that it is a staging File for the Pipeline. Gives the Jenkins Pipeline its Orders.

Be sure to update the following values within the Jenkins File with your own unique information,

pipeline {
    agent any
    environment {
        AWS_REGION = =='us-east-1'== 
    }
    stages {
        stage('Set AWS Credentials') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: =='JenkinsUser'== 
                ]]) {
                    sh '''
                    echo "AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID"
                    aws sts get-caller-identity
                    '''
                }
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/MarkofIT91/autoScale.git' ==<------ This URL==
            }
        }
        stage('Initialize Terraform') {
            steps {
                sh '''
                terraform init
                '''
            }
        }
        stage('Plan Terraform') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: '==JenkinsUser=='
                ]]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform plan -out=tfplan
                    '''
                }
            }
        }
        stage('Apply Terraform') {
            steps {
                input message: "Approve Terraform Apply?", ok: "Deploy"
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: '==JenkinsUser=='
                ]]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform apply -auto-approve tfplan
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Terraform deployment completed successfully!'
        }
        failure {
            echo 'Terraform deployment failed!'
        }
    }
}

"CredentialsID" - pulled from your Jenkins Global credentials (unrestricted) menu. ![[Pasted image 20250203175424.png]]
"AWS_REGION" - pulled from AWS Region and AZ you're currently operating your Jenkins IAM User account from.![[Pasted image 20250203175601.png]]
"git branch: 'main', url" - pulled from Github![[Pasted image 20250203160445.png]]
Run "git clone (insert repo URL pulled from GitHub here)" to copy the repo URL from GitHub to the folder you will be accessing the file from within your command line tool, eg. GitBash, Powershell etc.![[Pasted image 20250204180032.png]]

Install Terraform and AWS through the Docker CLI Terminal.
![[Pasted image 20250203210759.png]]


Run "docker ps" to check for running Containers. The first three characters of the Container ID are enough to identify it when running some code.

Install AWS CLI and Terraform into Docker Container Image

In GitBash, Powershell, Docker Terminal (Windows Powershell) or some other command line, Run "docker exec -it --user root (your Container name here) bash" in order to open and operate and run code from the Docker command line

Run "apt update && apt install -y awscli" to install or update AWS CLI within the Docker Container.

Run the following line one after another or all at once to install Terraform in the docker container if you haven't already,
![[Pasted image 20250203151058.png]]

Run "terraform --version" to verify Terraform Installation.

Next go back to your Jenkins "Dashboard"  and go into your created Pipeline and select "Build Now"
![[Pasted image 20250203202711.png]]
As the Pipeline is starting up select the Build that is being worked,
![[Pasted image 20250203202941.png]]

and select "Console Output"
![[Pasted image 20250203203058.png]]

If it runs correctly you should see something like this,![[Pasted image 20250203203503.png]]
![[Pasted image 20250203203425.png]]

==Should it Fail, Time to Troubleshoot!!==

==The process for Tear Down is as follows,==

![[Pasted image 20250203211055.png]]

Enter your Docker Jenkins Pipeline by running "docker exec -it --user root (first 3 numbers of your Container ID) bash".
"cd" into your Jenkins Pipeline by running "cd $JENKINS_HOME/workspace/(your Pipeline name here)"

![[Pasted image 20250204183712.png]]

run the following commands with your AWS Access key Information,

export AWS_ACCESS_KEY_ID="(AWS Access Key used for Your Jenkins Account)"
export AWS_SECRET_ACCESS_KEY="(AWS Secret Access Key used for Your Jenkins Account)"

Then run, "terraform destroy"
![[Pasted image 20250204185049.png]]
![[Pasted image 20250204185210.png]]
![[Pasted image 20250204185325.png]]
