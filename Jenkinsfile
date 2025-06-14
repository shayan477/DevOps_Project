pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        // Use your GitHub key to clone
        sshagent(credentials: ['github-ssh-key']) {
          git(
            url: 'git@github.com:shayan477/DevOps_Project.git',
            branch: 'main'
          )
        }
      }
    }

    stage('Terraform') {
      steps {
        // Inject your Azure SP creds so Terraform can talk to Azure
        withCredentials([
          usernamePassword(
            credentialsId: 'azure-sp-credentials',
            usernameVariable: 'ARM_CLIENT_ID',
            passwordVariable: 'ARM_CLIENT_SECRET'
          )
        ]) {
          // Set the two env vars Terraform also needs
          withEnv([
            "ARM_SUBSCRIPTION_ID=4bc05121-31f4-4433-9642-0cc1637e78d5",
            "ARM_TENANT_ID=903341ab-c6ca-4510-89cd-ac942bb328c6"
          ]) {
            dir('terraform') {
              sh 'terraform init'
              sh 'terraform apply -auto-approve'
            }
            script {
              // Capture the public IP output for the next stage
              env.PUBLIC_IP = sh(
                script: "terraform output -raw public_ip",
                returnStdout: true
              ).trim()
            }
          }
        }
      }
    }

    stage('Ansible Deploy') {
      steps {
        // Use your Jenkins-to-VM key for Ansible’s SSH
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
        sh "curl -I http://${PUBLIC_IP}"
      }
    }
  }
}

