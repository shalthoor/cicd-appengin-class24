pipeline {
    agent any

    environment {
        PROJECT_ID = 'october-2026-511906'
    }

    stages {

        stage('Setup Python 3.13') {
            steps {
                echo "===== Setting up Python 3.13 ====="
                sh '''
                    python3 --version
                    rm -rf venv
                    python3 -m venv venv

                    . venv/bin/activate
                    python --version
                    pip install --upgrade pip setuptools wheel
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Clean Workspace') {
            steps {
                echo "===== Cleaning old deployment artifacts ====="
                sh 'rm -rf .gcloudignore staging/'
            }
        }

        stage('Authenticate GCP') {
            steps {
                echo "===== Authenticating with GCP ====="
                withCredentials([file(credentialsId: 'gcp-service-account', variable: 'KEYFILE')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$KEYFILE
                        gcloud config set project ${PROJECT_ID}
                    '''
                }
            }
        }

        stage('Deploy to App Engine') {
            steps {
                echo "===== Deploying App Engine ====="
                sh '''
                    . venv/bin/activate
                    gcloud app deploy app.yaml --quiet --verbosity=info
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
        success {
            echo "Deployment succeeded!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
