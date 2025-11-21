pipeline {
  agent any

  environment {
    ANSIBLE_PLAYBOOK = 'playbooks/install-nginx.yml'
    ANSIBLE_INVENTORY = 'inventory/aws_ec2.yml'
    AWS_REGION = 'ap-south-1'
    PATH = "${env.HOME}/.local/bin:${env.PATH}"
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
          python3 -m pip install --user ansible boto3 botocore || true
          export PATH="$HOME/.local/bin:$PATH"
          ansible --version || true
        '''
      }
    }

    stage('Prepare Credentials') {
      steps {
        withCredentials([
          [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'],
          sshUserPrivateKey(credentialsId: 'jenkins-ssh-ec2', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')
        ]) {
          sh '''
            chmod 600 "$SSH_KEY" || true
            export AWS_DEFAULT_REGION=${AWS_REGION}
            export PATH="$HOME/.local/bin:$PATH"
            echo "Discovered hosts (debug):"
            ansible-inventory -i "${ANSIBLE_INVENTORY}" --list || true
          '''
        }
      }
    }

    stage('Run Ansible Playbook (Dynamic Inventory)') {
      steps {
        withCredentials([
          [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'],
          sshUserPrivateKey(credentialsId: 'jenkins-ssh-ec2', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')
        ]) {
          sh '''
            export PATH="$HOME/.local/bin:$PATH"
            export AWS_DEFAULT_REGION=${AWS_REGION}
            export ANSIBLE_HOST_KEY_CHECKING=False
            chmod 600 "$SSH_KEY" || true

            ansible-playbook -i "${ANSIBLE_INVENTORY}" "${ANSIBLE_PLAYBOOK}" \
              -u "$SSH_USER" --private-key="$SSH_KEY" -vv
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Dynamic-inventory Ansible run succeeded.'
    }
    failure {
      echo 'Dynamic-inventory Ansible run FAILED — check console output.'
    }
  }
}
