pipeline {
    agent any
     environment{
        ARTIFACTORY_LOGIN=credentials('artifactory-login')
    }
    stages {
        stage('Git checkout') {
            steps {
                echo "Git checkout"
                sh 'git --version'
                git branch: 'ansible', url: 'https://github.com/helijunky/ci-cd-test.git'
            }
        }
        stage('Execute ansible playbook') {
            steps {
                echo "Execute ansible playbook using plugin"
                sh "sudo sed -i -e 's/ARTIFACTORY_LOGIN_USR/${params.ARTIFACTORY_LOGIN_USR}/g' docker.yml'"
                sh "sudo sed -i -e 's/ARTIFACTORY_LOGIN_PSW/${params.ARTIFACTORY_LOGIN_PSW}/g' docker.yml'"
                ansiblePlaybook credentialsId: 'linux', disableHostKeyChecking: true, extras: '-v', installation: 'ansible', inventory: 'hosts', playbook: 'docker.yml'
            }
        }
    }
}
