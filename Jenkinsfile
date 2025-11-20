pipeline {
  agent any

  environment {
    ANSIBLE_PLAYBOOK = 'playbooks/install-nginx.yml'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare SSH Key') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-ssh-ec2',
                                          keyFileVariable: 'SSH_KEY',
                                          usernameVariable: 'SSH_USER')]) {
          sh '''
            echo "Using SSH key: $SSH_KEY"
            echo "SSH Username: $SSH_USER"

            # make key secure
            chmod 600 "$SSH_KEY"

            # Optional ansible check
            ansible --version || true
          '''
        }
      }
    }

    stage('Run Ansible Playbook') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-ssh-ec2',
                                          keyFileVariable: 'SSH_KEY',
                                          usernameVariable: 'SSH_USER')]) {
          sh '''
            export ANSIBLE_HOST_KEY_CHECKING=False
            export ANSIBLE_PRIVATE_KEY_FILE="$SSH_KEY"

            ansible-playbook "$ANSIBLE_PLAYBOOK" \
              -i inventory/hosts \
              -u "$SSH_USER" \
              --private-key="$SSH_KEY" \
              -vv
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Ansible playbook executed successfully!'
    }
    failure {
      echo 'Ansible run failed — check logs.'
    }
  }
}
