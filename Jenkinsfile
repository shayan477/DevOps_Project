pipeline {
  agent any

  // Prevent Jenkins from doing the default pipeline checkout
  options { skipDefaultCheckout() }

  stages {
    stage('Checkout') {
      steps {
        script {
          sshagent(['github-ssh-key']) {
            // SSH‐based clone of your repo
            git(
              url:    'git@github.com:shayan477/DevOps_Project.git',
              branch: 'main'
            )
          }
        }
      }
    }

    stage('Terraform') {
      steps {
        // Inject your Azure SP credentials into env vars
        withCredentials([usernamePassword(
          credentialsId: 'azure-sp-credentials',
          usernameVariable: 'ARM_CLIENT_ID',
          passwordVariable: 'ARM_CLIENT_SECRET'
        )]) {
          withEnv([
            "ARM_SUBSCRIPTION_ID=4bc05121-31f4-4433-9642-0cc1637e78d5",
            "ARM_TENANT_ID=903341ab-c6ca-4510-89cd-ac942bb328c6"
          ]) {
            dir('terraform') {
              sh '''
                set -o pipefail
                terraform init         \
                  | tee init.log
                terraform apply        \
                  -auto-approve         \
                  | tee apply.log
              '''
            }
            script {
              // Read the public_ip output for later stages
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
        script {
          sshagent(['jenkins-ssh-key']) {
            sh '''
              # Build a minimal inventory
              printf "[webserver]\n%s ansible_user=azureuser\n" "${PUBLIC_IP}" > inventory.ini

              # Run the playbook over the ssh-agent identity
              ansible-playbook \
                -i inventory.ini \
                ansible/install_web.yml
            '''
          }
        }
      }
    }

    stage('Verify') {
      steps {
        sh '''
          echo "=== HTTP HEAD for http://${PUBLIC_IP} ==="
          curl -I "http://${PUBLIC_IP}"
        '''
      }
    }
  }
}

