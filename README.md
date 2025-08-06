 [GitHub](https://github.com/sanketpatil461/nodepipeline)



To create a nodejs project in jenkins with pulling the code from git into the deployment server, building, testing, and deploying it on the deployment server

we installed nodejs tool and given a name to it we have to check it bcoz we have to write it in the script Dashboard ---- Manage Jebkins ----- Tools ------- Check the name of Nodejs

then we have to install the plugin for doing ssh into the live server Dashboard---Manage Jenkins----Plugin----Available Plugin----search 'SSh agent' and intall it

then we have to create a user for ssh Dashboard ----- Manage Jenkins ---- Credentials ---- Global ---- Add Credentials ----- Kind ----- SSH username with private key ----- Username as Ubuntu --- --- and in private key enter the .pem key of the deployment server ---- create

now create a directory in local (mkdir nodeproject) ----- Create package.json, index.js, test directory and test.js inside test directory ---- ----- push the package.json, index.js and test directory to github

getting ready Deployment server launch a new server and install npm, nodejs, and git in the server ------ create a directory (mynodeapp) and initialise git init

Now create a pipeline give name and description ---- Build Triggers ----> Github hook triggers ---- pipeline syntax ----- in sample step click on checkout from version control --- ---- and paste the git repository url ---- select the branch on which we pushed the code ----- Generate Pipeline Script ----- Copy the script ----------

below is the code of the whole pipeline ------>

    pipeline {
    agent any  // This allows Jenkins to run the pipeline on any available agent

    tools {
        nodejs 'mynodejs'  // Use the Node.js tool named 'mynodejs' configured in Jenkins
    }

    stages {

        stage('Git Clone') {
            steps {
                echo 'Cloning from Git...'
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[url: 'https://github.com/sanketpatil461/nodepipeline.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                echo 'Building Node.js project...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing Node.js project...'
                sh './node_modules/.bin/mocha --exit ./test/test.js'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to the server...'
                script {
                    // Use SSH Agent credentials to authenticate
                    sshagent(credentials: ['eb65d85a-ca49-4d9e-a14d-171dbcdded0a']) {
                        sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@3.92.192.219 << EOF
                        cd nodepipeline || git clone https://github.com/sanketpatil461/nodepipeline.git && cd nodepipeline
                        git pull origin master
                        npm install
                        sudo npm install -g pm2
                        pm2 restart index.js || pm2 start index.js
                        exit
                        EOF
                        '''
                    }
                }
            }
        }
    }
}
