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
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo snap install helm --classic'"
                //sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'helm repo add rocketchat-server https://rocketchat.github.io/helm-charts && helm repo update'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'helm repo add rocketchat-server https://bin.swisscom.com/artifactory/ptt-helm-local --username ${env.ARTIFACTORY_LOGIN_USR} --password ${env.ARTIFACTORY_LOGIN_PSW} && helm repo update'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'minikube start'"
                sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'minikube addons enable ingress'"
                sleep 10
            }
        }
        stage('Deploy with helm on server') {
            steps {
                script {
                    echo "Deploy with helm on server"
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'helm upgrade --install engel-rocketchat rocketchat-server/rocketchat --set mongodb.auth.password=abc1234567890 --set mongodb.auth.rootPassword=xyz0987654321 --set ingress.enabled=true'"
                    sleep 90
                    PROXY_URL = (sh(script: "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} \"kubectl get ingress | grep engel-rocketchat-rocketchat | awk -F ' ' '{print \$ 4}'\"", returnStdout: true)).trim()
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo sed -i \"/proxy_pass/c\\\\            proxy_pass http://$PROXY_URL:80/;\" /etc/nginx/sites-available/default'"
                    sh "ssh -o StrictHostKeyChecking=no -i ${env.SERVER_LOGIN} ${params.SSH_USER}@${params.SERVER_FQDN} 'sudo service nginx configtest && sudo service nginx restart'"
                    sleep 30
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
