pipeline {
  agent any

  environment {
    ANSIBLE_PLAYBOOK   = 'playbooks/install-nginx.yml'
    ANSIBLE_INVENTORY  = 'inventory/aws_ec2.yml'
    AWS_REGION         = 'ap-south-1'
    PATH               = "${env.HOME}/.local/bin:${env.PATH}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install deps') {
      steps {
        sh '''
          # install user-level ansible deps if needed (idempotent)
          python3 -m pip install --user ansible boto3 botocore || true
          export PATH="$HOME/.local/bin:$PATH"
          ansible --version || true
        '''
      }
    }

    stage('Prepare Credentials & Validate Inventory') {
      steps {
        // NOTE: replace REPLACE_WITH_YOUR_VAULT_CRED_ID with your actual Jenkins credential ID
        withCredentials([
          [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'],
          sshUserPrivateKey(credentialsId: 'jenkins-ssh-ec2', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER'),
          file(credentialsId: 'REPLACE_WITH_YOUR_VAULT_CRED_ID', variable: 'VAULT_FILE')
        ]) {
          sh '''
            chmod 600 "$SSH_KEY" || true
            export AWS_DEFAULT_REGION=${AWS_REGION}
            export PATH="$HOME/.local/bin:$PATH"

            echo "DEBUG: show inventory (for troubleshooting only)"
            ansible-inventory -i "${ANSIBLE_INVENTORY}" --list || true
          '''
        }
      }
    }

    stage('Run Ansible Playbook (Dynamic Inventory + Vault)') {
      steps {
        withCredentials([
          [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'],
          sshUserPrivateKey(credentialsId: 'jenkins-ssh-ec2', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER'),
          file(credentialsId: 'REPLACE_WITH_YOUR_VAULT_CRED_ID', variable: 'VAULT_FILE')
        ]) {
          sh '''
            export PATH="$HOME/.local/bin:$PATH"
            export AWS_DEFAULT_REGION=${AWS_REGION}
            export ANSIBLE_HOST_KEY_CHECKING=False
            chmod 600 "$SSH_KEY" || true

            # Run playbook with vault password file provided by Jenkins secret-file credential
            ansible-playbook -i "${ANSIBLE_INVENTORY}" "${ANSIBLE_PLAYBOOK}" \
              -u "$SSH_USER" --private-key="$SSH_KEY" --vault-password-file "$VAULT_FILE" -vv
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Dynamic-inventory Ansible run with Vault succeeded.'
    }
    failure {
      echo 'Dynamic-inventory Ansible run with Vault FAILED — check console output.'
    }
  }
}
