pipeline {
agent any

environment {
AZURE_SUB = credentials('azure-sp-credentials')
SSH_KEY = credentials('jenkins-ssh-key')
}

stages {
stage('Checkout') {
steps {
git credentialsId: 'github-ssh-key', branch: 'main', url: 'git@github.com:shayan477/DevOps_Project.git'
}
}

 stage('Terraform') {
  steps {
    withEnv([
      "ARM_SUBSCRIPTION_ID=${AZURE_SUB_USR}",
      "ARM_CLIENT_ID=${AZURE_SUB_PSW}",
      "ARM_CLIENT_SECRET=${AZURE_SUB_PSW}",
      "ARM_TENANT_ID=${AZURE_SUB_USR}"
    ]) {
      sh """
        cd terraform &&
        terraform init &&
        terraform apply -auto-approve \
          -var subscription_id=${AZURE_SUB_USR} \
          -var client_id=${AZURE_SUB_PSW} \
          -var client_secret=${AZURE_SUB_PSW} \
          -var tenant_id=${AZURE_SUB_USR}
      """

      script {
        env.PUBLIC_IP = sh(
          script: "terraform -chdir=terraform output -raw public_ip",
          returnStdout: true
        ).trim()
      }
    }
  }
}

stage('Ansible') {
  steps {
    sshagent(['jenkins-ssh-key']) {
      sh """cat > inventory.ini << 'EOL'
[webserver]
${env.PUBLIC_IP} ansible_user=azureuser ansible_ssh_private_key_file=$SSH_KEY
EOL"""
sh 'ansible-playbook -i inventory.ini ansible/install_web.yml'
}
}
}
}
}
EOF
