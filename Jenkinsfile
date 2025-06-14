pipeline {
agent any

environment {
ARM_SUBSCRIPTION_ID = 'your-subscription-id'
ARM_TENANT_ID = 'your-tenant-id'
}

stages {
stage('Checkout') {
steps {
sshagent(credentials: ['github-ssh-key']) {
git(
url: 'git@github.com:shayan477/DevOps_Project.git',
branch: 'main'
)
}
}
}stage('Terraform') {
  steps {
    withCredentials([
      usernamePassword(
        credentialsId: 'azure-sp-credentials',
        usernameVariable: 'ARM_CLIENT_ID',
        passwordVariable: 'ARM_CLIENT_SECRET'
      )
    ]) {
      dir('terraform') {
        sh '''
          terraform init
          terraform apply -auto-approve \
            -var subscription_id=$ARM_SUBSCRIPTION_ID \
            -var client_id=$ARM_CLIENT_ID \
            -var client_secret=$ARM_CLIENT_SECRET \
            -var tenant_id=$ARM_TENANT_ID
        '''
      }
      script {
        env.PUBLIC_IP = sh(
          script: "terraform -chdir=terraform output -raw public_ip",
          returnStdout: true
        ).trim()
      }
    }
  }
}

stage('Ansible Deploy') {
  steps {
    sshagent(credentials: ['jenkins-ssh-key']) {
      sh '''
        echo "[webserver]" > inventory.ini
        echo "${PUBLIC_IP} ansible_user=azureuser ansible_ssh_private_key_file=~/.ssh/jenkins_key" >> inventory.ini
        ansible-playbook -i inventory.ini ansible/install_web.yml
      '''
    }
  }
}

stage('Verify') {
  steps {
    sh 'curl -I http://${PUBLIC_IP}'
  }
}
}
}
