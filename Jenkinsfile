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

    stage('Prepare') {
      steps {
        // load SSH key from Jenkins credentials and write to file
        withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key',
                                          keyFileVariable: 'SSH_KEY',
                                          usernameVariable: 'UBUNTU')]) {
          sh '''
            echo "Using SSH key at $SSH_KEY"
            chmod 600 $SSH_KEY
            # Optional: show ansible version
            ansible --version || true
          '''
        }
      }
    }

    stage('Run Ansible Playbook') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key',
                                          keyFileVariable: 'SSH_KEY',
                                          usernameVariable: 'UBUNTU')]) {
          // Ensure ANSIBLE_HOST_KEY_CHECKING=FALSE or configure known_hosts properly
          sh '''
            export ANSIBLE_HOST_KEY_CHECKING=False
            export ANSIBLE_PRIVATE_KEY_FILE="$SSH_KEY"
            # If your ansible.cfg uses relative inventory path, run from repo root
            ansible-playbook ${ANSIBLE_PLAYBOOK} -i inventory/hosts -u $SSH_USER --private-key="$SSH_KEY" -vv
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Ansible run completed successfully.'
    }
    failure {
      echo 'Ansible run failed. Check logs.'
    }
  }
}
