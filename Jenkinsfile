pipeline {
    agent any

    parameters {
        string(name: 'TARGET_HOST', defaultValue: '', description: 'Ubuntu server IP')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Ansible') {
            steps {
                sh 'ansible --version'
            }
        }

        stage('Install Nginx') {
            steps {
                script {
                    if (!params.TARGET_HOST?.trim()) {
                        error('TARGET_HOST cannot be empty')
                    }
                }

                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    ),
                    string(
                        credentialsId: 'ubuntu-sudo-password',
                        variable: 'SUDO_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x

                        printf '[webservers]\\n%s\\n' "$TARGET_HOST" > inventory/jenkins-hosts.ini

                        printf '%s' "$SUDO_PASSWORD" > .ansible_become_pass
                        chmod 600 .ansible_become_pass

                        trap 'rm -f .ansible_become_pass inventory/jenkins-hosts.ini' EXIT

                        ansible-playbook \
                          -i inventory/jenkins-hosts.ini \
                          -u "$SSH_USER" \
                          --private-key "$SSH_KEY" \
                          playbooks/install-nginx.yml \
                          --become \
                          --become-password-file .ansible_become_pass
                    '''
                }
            }
        }
    }
}
