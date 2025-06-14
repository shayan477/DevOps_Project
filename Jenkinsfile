pipeline {
  agent any

  environment {
    # Loads your Azure SP values from Jenkins credentials:
    ARM_SUBSCRIPTION_ID = credentials('azure-sp-credentials_USR')
    ARM_CLIENT_ID       = credentials('azure-sp-credentials_USR')
    ARM_CLIENT_SECRET   = credentials('azure-sp-credentials_PSW')
    ARM_TENANT_ID       = credentials('azure-sp-credentials_USR')
  }

  stages {
    stage('Checkout') {
      steps {
        git(
          credentialsId: 'github-ssh-key',
          branch:        'main',
          url:           'git@github.com:shayan477/DevOps_Project.git'
        )
      }
    }

    stage('Terraform') {
      steps {
        dir('terraform') {
          sh '''
            terraform init
            terraform apply -auto-approve
          '''
        }
        script {
          // Capture the public_ip output for later
          env.PUBLIC_IP = sh(
            script: "terraform -chdir=terraform output -raw public_ip",
            returnStdout: true
          ).trim()
        }
      }
    }

    stage('Ansible Deploy') {
      steps {
        sshagent(['jenkins-ssh-key']) {
          sh '''
            echo "[webserver]" > inventory.ini
            echo "${PUBLIC_IP} ansible_user=azureuser ansible_ssh_private_key_file=$SSH_KEY" >> inventory.ini
            ansible-playbook -i inventory.ini ansible/install_web.yml
          '''
        }
      }
    }

    stage('Verify') {
      steps {
        sh "curl -I http://${PUBLIC_IP}"
      }
    }
  }
}

