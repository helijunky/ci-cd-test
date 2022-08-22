pipeline {
    agent any
    environment{
        ARTIFACTORY_LOGIN=credentials('artifactory-login')
        SERVER_LOGIN=credentials('linux')
    }
    parameters {
        choice(name: 'BASE_INSTALLATION',
            choices: ['NO','WITH_BASE_INSTALLATION'],
            description: 'Choose to install the base installation on the server')

        choice(name: 'MONGODB',
            choices: ['NO','REDEPLOY_MONGODB'],
            description: 'Choose to redeploy the mongodb on the server (loose user data)')

        string(name: 'ARTIFACTORY_URL',
               defaultValue: 'ptt-docker-local.bin.swisscom.com',
               description: 'Artifactory URL')

        string(name: 'SSH_USER',
               defaultValue: 'ubuntu',
               description: 'user for ssh connection')

        string(name: 'SERVER_FQDN',
               defaultValue: 'ec2-18-193-120-77.eu-central-1.compute.amazonaws.com',
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
                sh "docker tag rocket.chat ${params.ARTIFACTORY_URL}/rocket.chat:4.8.1"
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
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo wget -O /etc/nginx/sites-available/default https://raw.githubusercontent.com/helijunky/ci-cd-test/k8s/nginx.conf'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo sed -i -e 's/SERVER_FQDN/${params.SERVER_FQDN}/g' /etc/nginx/sites-available/default'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/certificate.key -out /etc/nginx/certificate.crt -subj \"/C=CH/ST=BE/L=Bern/O=PTT/OU=Engel/CN=${params.SERVER_FQDN}/emailAddress=ptt@engel.com\"'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo chmod 400 /etc/nginx/certificate.key'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo openssl dhparam -out /etc/nginx/dhparams.pem 2048'"
                //sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo service nginx configtest && sudo service nginx restart'"

                echo "Install and start Minikube"
                catchError {
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'minikube delete'"
                }
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo dpkg -i minikube_latest_amd64.deb'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'curl -LO \"https://dl.k8s.io/release/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl\"'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'curl -LO \"https://dl.k8s.io/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256\"'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo systemctl enable docker && sudo systemctl start docker'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'minikube start'"
            }
        }
        stage('Deploy with k8s on server') {
            steps {
                script {
                    echo "Deploy with k8s on server"
                    if (params.MONGODB == 'REDEPLOY_MONGODB') {
                        sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'kubectl apply -f https://raw.githubusercontent.com/helijunky/ci-cd-test/k8s/mongo.yaml'"
                        sleep 90
                        sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'kubectl exec -it rocketmongo-0 -- mongo --eval \"printjson(rs.initiate())\"'"
                    }
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'kubectl apply -f https://raw.githubusercontent.com/helijunky/ci-cd-test/k8s/rocketchat.yaml'"
                    sleep 90
                    PROXY_URL = (sh(script: "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'minikube service rocketchat-server --url'", returnStdout: true)).trim()
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo sed -i \"/proxy_pass/c\\\\            proxy_pass $PROXY_URL/;\" /etc/nginx/sites-available/default'"
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo service nginx configtest && sudo service nginx restart'"
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
