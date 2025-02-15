
def COMMIT
def BRANCH_NAME
def GIT_BRANCH
pipeline
{
 agent any
 environment
    {
     AWS_ACCOUNT_ID="245218175547"
     AWS_DEFAULT_REGION="ap-south-1" 
     IMAGE_REPO_NAME="mavenwebapp"
     REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
    tools
    {
      maven 'Mevan'
    }   

    options 
    {
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '4', daysToKeepStr: '', numToKeepStr: '4')
    timestamps()
    }
    stages
    {
        stage('Code checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/development']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/VOpandit/JAVA-CI-CD-PIPELINE.git']]])
                    COMMIT = sh (script: "git rev-parse --short=10 HEAD", returnStdout: true).trim()  
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Execute Sonarqube Report') {
            steps {
                withSonarQubeEnv('Sonarqube-Server') {
                    sh "mvn sonar:sonar"
                }  
            }
        }
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true, credentialsId: 'SONARQUBE-CRED'
                }
            }
        }
        stage('Login to AWS ECR') {
            steps {
                script {
                    sh "/usr/local/bin/aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
            }
        }
        stage('Building Docker Image') {
            steps {
                script {
                    sh "docker build . -t ${REPOSITORY_URI}:mavenwebapp-${COMMIT}"
                }
            }
        }
        stage('Pushing Docker image into ECR') {
            steps {
                script {
                    sh "docker push ${REPOSITORY_URI}:mavenwebapp-${COMMIT}"
                }
            }
        }
        stage('Update image in K8s manifest file') {
            steps {
                sh """#!/bin/bash
                sed -i 's/VERSION/$COMMIT/g' deployment.yaml
                """
            }
        }
     
        stage('Deploy to K8s cluster') {
            steps {
                sh 'kubectl apply -f deployment.yaml --record=true'
                sh """#!/bin/bash
                sed -i 's/$COMMIT/VERSION/g' deployment.yaml
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            slackSend channel: 'build-notifications',color: 'good', message: "started  JOB : ${env.JOB_NAME}  with BUILD NUMBER : ${env.BUILD_NUMBER}  BUILD_STATUS: - ${currentBuild.currentResult} To view the dashboard (<${env.BUILD_URL}|Open>)"
            emailext attachLog: true, body: '''BUILD IS SUCCESSFULL - $PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
 
            Check console output at $BUILD_URL to view the results.
 
            Regards,
 
            TAASHEE LINUX SERVICE
            ''', compressLog: true, replyTo: '<Your mail id>', 
            subject: '$PROJECT_NAME - $BUILD_NUMBER - $BUILD_STATUS', to: 'njdevops321@gmail.com'
        }
        failure
        {
            slackSend channel: 'build-notifications',color: 'danger', message: "started  JOB : ${env.JOB_NAME}  with BUILD NUMBER : ${env.BUILD_NUMBER}  BUILD_STATUS: - ${currentBuild.currentResult} To view the dashboard (<${env.BUILD_URL}|Open>)"
            emailext attachLog: true, body: '''BUILD IS FAILED - $PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
 
            Check console output at $BUILD_URL to view the results.
 
            Regards,
 
            Taashee Linux Service
            ''', compressLog: true, replyTo: '<Your mail id>', 
            subject: '$PROJECT_NAME - $BUILD_NUMBER - $BUILD_STATUS', to: '<Your mail id>'
        }
    }

}
