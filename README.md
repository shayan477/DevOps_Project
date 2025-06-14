# DevOps Project

## Structure

  - terraform/      → Azure VM provisioning
  - ansible/        → Apache install & site deploy
  - app/index.html  → Static website
  - Jenkinsfile     → CI pipeline

## Next Steps

1. In `terraform/variables.tf` fill in:
   - subscription_id
   - client_id
   - client_secret
   - tenant_id

2. Push this repo to GitHub (or Azure Repos).

3. In Jenkins:
   - Add **azure-sp-credentials** (your Azure Service Principal)
   - Add **jenkins-ssh-key** (private key from `~/.ssh/jenkins_key`)
   - Create a **Pipeline** pointing to your repo.

4. Trigger a build! 🎉
