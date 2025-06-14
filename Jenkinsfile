pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        sshagent(credentials: ['github-ssh-key']) {
          git(
            url:    'git@github.com:shayan477/DevOps_Project.git',
            branch: 'main'
          )
        }
      }
    }

    stage('Terraform') {
      steps {
        // Inject your Azure SP credentials
        withCredentials([usernamePassword(
          credentialsId: 'azure-sp-credentials',
          usernameVariable: 'ARM_CLIENT_ID',
          passwordVariable: 'ARM_CLIENT_SECRET'
        )]) {
          // Export static subscription & tenant IDs as env vars
          withEnv([
            "ARM_SUBSCRIPTION_ID=4bc05121-31f4-4433-9642-0cc1637e78d5",
            "ARM_TENANT_ID=903341ab-c6ca-4510-89cd-ac942bb328c6"
          ]) {
            // Run Terraform in its folder
            dir('terraform') {
              sh '''
                terraform init
                terraform apply -auto-approve
              '''
            }
            // Capture the VM public IP for downstream stages
            script {
              env.PUBLIC_IP = sh(
                script: "terraform -chdir=terraform output -raw public_ip",
                returnStdout: true
              ).trim()
            }
          }
        }
      }
    }

    stage('Ansible Deploy') {
      steps {
        // Use SSH agent with the VM key to connect
        sshagent(credentials: ['jenkins-ssh-key']) {
          sh '''
            echo "[webserver]" > inventory.ini
            echo "${PUBLIC_IP} ansible_user=azureuser" >> inventory.ini
            ansible-playbook -i inventory.ini ansible/install_web.yml
          '''
        }
      }
    }

    stage('Verify') {
      steps {
        // Quick HTTP HEAD check to ensure Apache is up
        sh "curl -I http://${PUBLIC_IP}"
      }
    }
  }
}

