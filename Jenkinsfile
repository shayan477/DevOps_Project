pipeline {
  agent any

  environment {
    // these four get pulled in from the azure-sp-credentials in the Terraform stage
    ARM_SUBSCRIPTION_ID = '4bc05121-31f4-4433-9642-0cc1637e78d5'
    ARM_TENANT_ID       = '903341ab-c6ca-4510-89cd-ac942bb328c6'
  }

  stages {
    stage('Checkout') {
      steps {
        // use your GitHub SSH key to check out
        sshagent(credentials: ['github-ssh-key']) {
          git url: 'git@github.com:shayan477/DevOps_Project.git', branch: 'main'
        }
      }
    }

    stage('Terraform Apply') {
      steps {
        // inject the SP credentials into ARM_CLIENT_ID/ARM_CLIENT_SECRET
        withCredentials([usernamePassword(
          credentialsId: 'azure-sp-credentials',
          usernameVariable: 'ARM_CLIENT_ID',
          passwordVariable: 'ARM_CLIENT_SECRET'
        )]) {
          dir('terraform') {
            sh '''
              terraform init
              terraform apply -auto-approve \
                -var="subscription_id=${ARM_SUBSCRIPTION_ID}" \
                -var="tenant_id=${ARM_TENANT_ID}" \
                -var="client_id=${ARM_CLIENT_ID}" \
                -var="client_secret=${ARM_CLIENT_SECRET}"
            '''
          }
          // capture the public IP for downstream stages
          script {
            env.PUBLIC_IP = sh(
              script: 'terraform -chdir=terraform output -raw public_ip',
              returnStdout: true
            ).trim()
          }
        }
      }
    }

    stage('Ansible Deploy') {
      steps {
        // use your Jenkins→VM SSH key to reach the newly-created VM
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
        // smoke-test that the web page is up
        sh "curl -I http://${PUBLIC_IP} | head -n 1"
      }
    }
  }

  post {
    success {
      echo "🎉 Pipeline complete! Visit http://${PUBLIC_IP}"
    }
    failure {
      echo "❌ Pipeline failed. Check the Console Output for details."
    }
  }
}

