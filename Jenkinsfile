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
