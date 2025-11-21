pipeline {
  agent any

  environment {
    ANSIBLE_PLAYBOOK = 'playbooks/site.yml'
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

    stage('Install deps & roles') {
      steps {
        sh '''
          python3 -m pip install --user ansible boto3 botocore || true
          export PATH="$HOME/.local/bin:$PATH"
          ansible --version || true

          # If you use requirements.yml, install roles into roles/ dir
          if [ -f requirements.yml ]; then
            ansible-galaxy install -r requirements.yml -p ./roles
          fi
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
            ansible-inventory -i "${ANSIBLE_INVENTORY}" --list || true
          '''
        }
      }
    }

    stage('Run Playbook (roles)') {
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

            # Run playbook that uses roles
            ansible-playbook -i "${ANSIBLE_INVENTORY}" "${ANSIBLE_PLAYBOOK}" \
              -u "$SSH_USER" --private-key="$SSH_KEY" -vv
          '''
        }
      }
    }
  }

  post {
    success { echo 'Roles-based Ansible run succeeded.' }
    failure { echo 'Roles-based Ansible run FAILED — check logs.' }
  }
}
