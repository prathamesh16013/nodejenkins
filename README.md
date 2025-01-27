#pipeline-code 



pipeline {
    agent any
     tools{
         nodejs 'mynode'
     }
     stages {
        stage('Git cloning') {
            steps {
                echo 'github checkout'
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: '<repolink>']])
            }
        }
        stage('Build') {
            steps {
                echo 'Building Nodejs-app'
                sh "npm install"
            }
        }
        stage('Test') {
            steps {
                echo 'Testing'
                sh "./node_modules/mocha/bin/_mocha --exit ./test/test.js"
            }
        }
        stage('Deploy'){
            steps{
                echo "Deploying"
                script{
                    sshagent(['credential-id']){
                        sh '''
                           ssh -o StrictHostKeyChecking=no ubuntu@<live-serverip><<EOF
                           cd /home/ubuntu/deploypipe/
                            git pull <repolink>
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
