pipeline {
    agent any
    environment{
        ARTIFACTORY_LOGIN=credentials('artifactory-login')
        SERVER_LOGIN=credentials('linux')
    }
    parameters {
        choice(name: 'BASE_INSTALLATION',
            choices: ['WITH_BASE_INSTALLATION', 'NO'],
            description: 'Choose to install the base installation on the server')

        choice(name: 'MONGODB',
            choices: ['REDEPLOY_MONGODB', 'NO'],
            description: 'Choose to redeploy the mongodb on the server (loose user data)')

        string(name: 'ARTIFACTORY_URL',
               defaultValue: 'ptt-docker-local.bin.swisscom.com',
               description: 'Artifactory URL')

        string(name: 'SSH_USER',
               defaultValue: 'ubuntu',
               description: 'user for ssh connection')

        string(name: 'SERVER_FQDN',
               defaultValue: 'ec2-3-73-38-212.eu-central-1.compute.amazonaws.com',
               description: 'Server address for ssh connection')
    }
    
    stages {
        stage('Git checkout') {
            steps {
                echo "Git checkout (for demonstration only)"
                sh 'git --version'
                git branch: 'main', url: 'https://github.com/helijunky/ci-cd-test.git'
            }
        }
        stage('Pull Docker images') {
            steps {
                echo "Build Docker image ${params.IMAGE}"
                sh "docker pull mongo:5.0"
                sh "docker tag mongo:5.0 ${params.ARTIFACTORY_URL}/mongo:5.0"
                sh "docker pull rocket.chat:4.8.1"
                sh "docker tag rocket.chat:4.8.1 ${params.ARTIFACTORY_URL}/rocket.chat:4.8.1"
            }
        }
        stage('Push Docker images to jFrog') {
            steps {
                echo "Push Docker image to jFrog"
                sh "docker login ${params.ARTIFACTORY_URL} --username ${env.ARTIFACTORY_LOGIN_USR} --password ${env.ARTIFACTORY_LOGIN_PSW}"
                sh "docker push ${params.ARTIFACTORY_URL}/mongo:5.0"
                sh "docker push ${params.ARTIFACTORY_URL}/rocket.chat:4.8.1"
                sh "docker logout ${params.ARTIFACTORY_URL}"
            }
        }
        stage('Base installation') {
            when {
                // Only deploy if the base installation is 'WITH_BASE_INSTALLATION'
                expression { params.BASE_INSTALLATION == 'WITH_BASE_INSTALLATION' }
            }
            steps {
                echo "Base installation on server"
                catchError {
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo apt-get update && sudo apt-get install ca-certificates curl gnupg lsb-release nginx -y'"
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo mkdir -p /etc/apt/keyrings && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --batch --yes --dearmor -o /etc/apt/keyrings/docker.gpg'"
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'echo \"deb [arch=\$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \$(lsb_release -cs) stable\" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null'"
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y && sudo chmod 666 /var/run/docker.sock'"
                }
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo wget -O /etc/nginx/sites-available/default https://raw.githubusercontent.com/helijunky/ci-cd-test/aws/nginx.conf'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo sed -i -e 's/SERVER_FQDN/${params.SERVER_FQDN}/g' /etc/nginx/sites-available/default'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/certificate.key -out /etc/nginx/certificate.crt -subj \"/C=CH/ST=BE/L=Bern/O=PTT/OU=Engel/CN=${params.SERVER_FQDN}/emailAddress=ptt@engel.com\"'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo chmod 400 /etc/nginx/certificate.key'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo openssl dhparam -out /etc/nginx/dhparams.pem 2048'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo service nginx configtest && sudo service nginx restart'"
            }
        }
        stage('Deploy on server') {
            steps {
                script {
                    echo "Deploy on server"
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker login ${params.ARTIFACTORY_URL} --username ${env.ARTIFACTORY_LOGIN_USR} --password ${env.ARTIFACTORY_LOGIN_PSW}'"
                    if (params.MONGODB == 'REDEPLOY_MONGODB') {
                        sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker pull ${params.ARTIFACTORY_URL}/mongo:5.0'"
                    }
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker pull ${params.ARTIFACTORY_URL}/rocket.chat:4.8.1'"
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker logout ${params.ARTIFACTORY_URL}'"
                }
            }
        }
        stage('Stop previous instance on server') {
            steps {
                script {
                    echo "Stopping previous instances on server"
                    catchError {
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker stop rocketchat'"
                    }
                    if (params.MONGODB == 'REDEPLOY_MONGODB') {
                        catchError {
                            sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker stop db'"
                        }
                    }
                }
            }
        }
        stage('Start new instance on server') {
            steps {
                script {
                    echo "Starting new images Server"
                    if (params.MONGODB == 'REDEPLOY_MONGODB') {
                        sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker run --rm --name db -d ${params.ARTIFACTORY_URL}/mongo:5.0 --replSet rs0 --oplogSize 128'"
                        sleep 5
                        sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker exec -i db mongo --eval \"printjson(rs.initiate())\"'"
                    }
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'docker run --rm --name rocketchat -p 3000:3000 --link db --env ROOT_URL=http://localhost --env MONGO_OPLOG_URL=mongodb://db:27017/local -d ${params.ARTIFACTORY_URL}/rocket.chat:4.8.1'"
                sleep 60
                }
            }
        }
        stage('Function test') {
            steps {
                echo "Function test"
                sh "wget --no-check-certificate https://${params.SERVER_FQDN}/"
                sh "rm index.html"

            }
        }
    }
}
