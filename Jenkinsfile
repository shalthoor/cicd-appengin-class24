```groovy
pipeline {
    agent any

    environment {
        PROJECT_ID = 'first-cicd-project'
        GCP_CREDENTIALS_ID = 'gcp-service-account'
    }

    stages {

        stage('Setup Python 3.13') {
            steps {
                echo '===== Setting up Python environment ====='

                sh '''
                    set -eu

                    python3 --version
                    rm -rf venv
                    python3 -m venv venv

                    . venv/bin/activate

                    python --version
                    python -m pip install --upgrade pip setuptools wheel
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Clean Deployment Artifacts') {
            steps {
                echo '===== Cleaning old deployment artifacts ====='

                sh '''
                    set -eu
                    rm -rf staging/
                '''
            }
        }

        stage('Authenticate GCP') {
            steps {
                echo '===== Authenticating with Google Cloud ====='

                withCredentials([
                    file(
                        credentialsId: "${GCP_CREDENTIALS_ID}",
                        variable: 'KEYFILE'
                    )
                ]) {
                    sh '''
                        set -eu

                        gcloud auth activate-service-account \
                            --key-file="$KEYFILE" \
                            --project="$PROJECT_ID"

                        gcloud config set project "$PROJECT_ID"

                        echo "===== Verify active account ====="
                        gcloud auth list

                        echo "===== Verify project ====="
                        gcloud config get-value project

                        echo "===== Verify App Engine application ====="
                        gcloud app describe \
                            --project="$PROJECT_ID"
                    '''
                }
            }
        }

        stage('Deploy to App Engine') {
            steps {
                echo '===== Deploying application to App Engine ====='

                withCredentials([
                    file(
                        credentialsId: "${GCP_CREDENTIALS_ID}",
                        variable: 'KEYFILE'
                    )
                ]) {
                    sh '''
                        set -eu

                        . venv/bin/activate

                        gcloud auth activate-service-account \
                            --key-file="$KEYFILE" \
                            --project="$PROJECT_ID"

                        gcloud app deploy app.yaml \
                            --project="$PROJECT_ID" \
                            --quiet \
                            --verbosity=info
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded!'
        }

        failure {
            echo 'Deployment failed. Review the stage logs above.'
        }

        always {
            echo '===== Cleaning Jenkins workspace ====='
            cleanWs()
        }
    }
}
```
