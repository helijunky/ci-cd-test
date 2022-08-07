pipeline {
    agent any
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
                ansiblePlaybook credentialsId: 'linux', disableHostKeyChecking: true, extras: '-v', installation: 'ansible', inventory: 'hosts', playbook: 'simple-test.yml'
            }
        }
    }
}
