pipeline {
    agent any
    environment{
        ARTIFACTORY_LOGIN=credentials('artifactory-login')
        SERVER_LOGIN=credentials('linux')
    }
    parameters {
        string(name: 'IMAGE',
               defaultValue: 'hello',
               description: 'Docker Image Name')

        string(name: 'ARTIFACTORY_URL',
               defaultValue: 'ptt-docker-local.bin.swisscom.com',
               description: 'Artifactory URL')

        string(name: 'SERVER_URL',
               defaultValue: 'ubuntu@ec2-18-196-35-245.eu-central-1.compute.amazonaws.com',
               description: 'user@server Address for ssh connection')
    }
    
    stages {
        stage('Git checkout') {
            steps {
                echo "Git checkout"
                sh 'git --version'
                git branch: 'main', url: 'https://github.com/helijunky/ci-cd-test.git'
            }
        }
        stage('Build Docker image') {
            steps {
                echo "Build Docker image ${params.IMAGE}"
                sh "docker build -t ${params.ARTIFACTORY_URL}/${params.IMAGE} ."
                sh "docker run --rm ${params.ARTIFACTORY_URL}/${params.IMAGE}"
            }
        }
        stage('Push Docker image to jFrog') {
            steps {
                echo "Push Docker image to jFrog"
                sh "docker login ${params.ARTIFACTORY_URL} --username ${env.ARTIFACTORY_LOGIN_USR} --password ${env.ARTIFACTORY_LOGIN_PSW}"
                sh "docker push ${params.ARTIFACTORY_URL}/${params.IMAGE}"
                sh "docker logout ${params.ARTIFACTORY_URL}"
            }
        }
        stage('Deploy on Server') {
            steps {
                echo "Deploy on Server"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SERVER_URL} 'uptime'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SERVER_URL} 'docker login ${params.ARTIFACTORY_URL} --username ${env.ARTIFACTORY_LOGIN_USR} --password ${env.ARTIFACTORY_LOGIN_PSW}'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SERVER_URL} 'docker pull ${params.ARTIFACTORY_URL}/${params.IMAGE}'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SERVER_URL} 'docker logout ${params.ARTIFACTORY_URL}'"
            }
        }
        stage('Start on Server') {
            steps {
                echo "Start on Server"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SERVER_URL} 'docker run --rm ${params.ARTIFACTORY_URL}/${params.IMAGE}'"
            }
        }
        stage('Function Test') {
            steps {
                echo "Function Test"
            }
        }
    }
}
