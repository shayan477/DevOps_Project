pipeline {
  agent any

  options { skipDefaultCheckout() }

  stages {
    stage('Checkout') {
      steps {
        script {
          wrap([$class: 'SSHAgent', credentials: ['github-ssh-key']]) {
            git url: 'git@github.com:shayan477/DevOps_Project.git', branch: 'main'
          }
        }
      }
    }

    stage('Terraform') {
      steps {
        withCredentials([ usernamePassword(
          credentialsId: 'azure-sp-credentials',
          usernameVariable: 'ARM_CLIENT_ID',
          passwordVariable: 'ARM_CLIENT_SECRET'
        ) ]) {
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
          wrap([$class: 'SSHAgent', credentials: ['jenkins-ssh-key']]) {
            sh '''
              echo "[webserver]" > inventory.ini
              echo "${PUBLIC_IP} ansible_user=azureuser" >> inventory.ini
              ansible-playbook -i inventory.ini ansible/install_web.yml
            '''
          }
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

