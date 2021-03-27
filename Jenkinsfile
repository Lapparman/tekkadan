#!/usr/bin/env groovy

pipeline {
    
    agent any
        
    stages {
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                sh 'docker build -t kovhico/landingpage:$BUILD_NUMBER .'
            }
        }
        stage('Build Docker Image') {
            when {
                tag 'stage'
            }
            steps {
                sh 'docker build -t kovhico/landingpage-stg:$BUILD_NUMBER .'
            }
        }
        stage('Push Image to Docker hub') {
            when {
                branch 'master'
            }
            steps {
                // withCredentials([string(credentialsId: 'docker_pwd', variable: 'dockerHubPwd')]) {
                //     sh "docker login -u mcpidinfra -p ${dockerHubPwd}"
                // checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[url: git@github.com/vhico07/landing-page]], branches: [[name: 'stage@1']]], poll: false
                sh 'docker push kovhico/landingpage-stg:$BUILD_NUMBER'
            }
        }
        stage('Deploy to Server') {
            agent { node { label 'worker' } }
            when {
                branch 'master'
            }
            steps {
                checkout scm
                sh """
                sed -i 's/latest/$BUILD_NUMBER/g' docker-compose.yml
                docker-compose up -d
                """
            }      
        }
        stage('Remove docker image last build') {
            when {
                branch 'master'
            }
            steps {
                sh 'docker rmi kovhico/landingpage:$BUILD_NUMBER'
            }      
        }
        stage('Git') {
            steps {
                step([$class: 'WsCleanup'])
                checkout scm
            }
        }        
    }
    post {
        success {
            slackSend channel: "jenkins", tokenCredentialId: "slack-arvy", color: "good", message: "*SUCCESS*\n Job: *${env.JOB_NAME}* build no.${env.BUILD_NUMBER} \n Environment: `${env.BRANCH_NAME}`\n by ${env.AUTHOR_NAME}\n More info at: ${env.BUILD_URL}"
        }
        failure {
            slackSend channel: "jenkins", tokenCredentialId: "slack-arvy", color: "danger", message: "*SUCCESS*\n Job: *${env.JOB_NAME}* build no.${env.BUILD_NUMBER} \n Environment: `${env.BRANCH_NAME}`\n by ${env.AUTHOR_NAME}\n More info at: ${env.BUILD_URL}"
        }
    }      
    // post {
    //   always {
    //     echo 'Cleaning up workspace'
    //     deleteDir()
    //     script {
    //       if (env.BRANCH_NAME == 'master') {
    //         def channel="jenkins"
    //         def cred="slack-arvy"
    //         slackNotifier(currentBuild.currentResult,channel,cred)
    //       } else {
    //         echo "No BRANCH specified!"
    //       }
    //     }
    //   }
    // }
}
