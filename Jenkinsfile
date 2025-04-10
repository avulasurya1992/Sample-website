/*
pipeline {
    agent { label 'node1' }

    triggers {
        githubPush()
    }

    environment {
        APP_DIR = "/var/www/html"   // standard Apache root on Linux
    }

    stages {
        
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/avulasurya1992/Sample-website.git'
            }
        }

        stage('Install Apache HTTPD') {
            steps {
                script {
                    sh '''
                    if ! which httpd > /dev/null 2>&1 && ! which apache2 > /dev/null 2>&1; then
                        echo "Installing Apache..."
                        if [ -f /etc/debian_version ]; then
                            sudo apt-get update
                            sudo apt-get install -y apache2
                            sudo systemctl start apache2
                            sudo systemctl enable apache2
                        elif [ -f /etc/redhat-release ]; then
                            sudo yum install -y httpd
                            sudo systemctl start httpd
                            sudo systemctl enable httpd
                        fi
                    else
                        echo "Apache already installed."
                    fi
                    '''
                }
            }
        }

        stage('Deploy Website') {
            steps {
                script {
                    sh '''
                    echo "Deploying website..."
                    sudo rm -rf ${APP_DIR}/*
                    sudo cp -r * ${APP_DIR}/
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh '''
                    echo "Checking deployed website..."
                    curl -I http://localhost || true
                    '''
                }
            }
        }
    }
}
*/

pipeline {
    agent { label 'node1' }

//    triggers {
//        githubPush()
//    }

    stages {
        stage('Prepare Node') {
            steps {
                echo 'Preparing Node - Installing Git...'
                sh '''
                    if ! which git > /dev/null 2>&1; then
                        echo "Installing Git..."
                        sudo dnf install -y git
                    else
                        echo "Git already installed."
                    fi
                '''
            }
        }

        stage('Checkout Application Code') {
            steps {
                git branch: 'main', url: 'https://github.com/avulasurya1992/Sample-website.git'
            }
        }

        stage('Prepare Environment') {
            steps {
                echo 'Installing Python and Ansible on node1...'
                sh '''
                    sudo dnf install -y python3 python3-pip
                    pip3 install boto3 botocore
                    sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
                    sudo dnf install -y ansible
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY'),
                    sshUserPrivateKey(credentialsId: 'web-app-server-key', keyFileVariable: 'SSH_KEY')
                ]) {
                    echo 'Running Ansible Playbook...'
                    sh '''
                        export ANSIBLE_HOST_KEY_CHECKING=False

                        # Clone the Ansible repo here
                        rm -rf ansible-task || true
                        git clone https://github.com/avulasurya1992/Ansible_practice_yaml_files.git ansible-task

                        # Change into the folder and run the playbook
                        cd ansible-task
                        ansible-playbook ec2-webapp-pipeline.yml --private-key=$SSH_KEY
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully on node1!'
        }
        failure {
            echo 'Deployment failed on node1.'
        }
    }
}
