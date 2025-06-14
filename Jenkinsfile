pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        sshagent(credentials: ['github-ssh-key']) {
          // Use SSH checkout so we don’t hit rate limits
          git(
            url: 'git@github.com:shayan477/DevOps_Project.git',
            branch: 'main'
          )
        }
      }
    }

    stage('Terraform') {
      steps {
        // Inject Azure SP client ID/secret
        withCredentials([
          usernamePassword(
            credentialsId: 'azure-sp-credentials',
            usernameVariable: 'ARM_CLIENT_ID',
            passwordVariable: 'ARM_CLIENT_SECRET'
          )
        ]) {
          // Export subscription & tenant from hard‑coded Jenkins env (set these in Build > Configure > Pipeline > Environment Variables)
          withEnv([
            "ARM_SUBSCRIPTION_ID=4bc05121-31f4-4433-9642-0cc1637e78d5",
            "ARM_TENANT_ID=903341ab-c6ca-4510-89cd-ac942bb328c6"
          ]) {
            dir('terraform') {
              sh '''
                terraform init
                terraform apply -auto-approve
              '''
            }
            script {
              // Capture the public IP for later stages
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
        // Use the same SSH key to connect to the VM
        sshagent(credentials: ['jenkins-ssh-key']) {
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

