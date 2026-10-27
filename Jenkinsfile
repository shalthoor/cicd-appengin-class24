pipeline {
    agent any

    environment {
        PROJECT_ID = 'first-cicd-project'
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

        withCredentials([
            file(
                credentialsId: 'gcp-service-account',
                variable: 'KEYFILE'
            )
        ]) {
            sh '''
                set -eu

                gcloud auth activate-service-account \
                    --key-file="$KEYFILE"

                gcloud config set project first-cicd-project

                echo "===== Verify authenticated account ====="
                gcloud auth list

                echo "===== Verify target App Engine application ====="
                gcloud app describe \
                    --project=first-cicd-project
            '''
        }
    }
}

stage('Deploy to App Engine') {
    steps {
        echo "===== Deploying to first-cicd-project ====="

        sh '''
            set -eu

            . venv/bin/activate

            gcloud app deploy app.yaml \
                --project=first-cicd-project \
                --quiet \
                --verbosity=info
        '''
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
