pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 45, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        PROJECT_ID = 'first-cicd-project'
        GCP_CREDENTIALS_ID = 'gcp-service-account'
        PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION = 'python'
    }

    stages {
        stage('Setup Python 3.13') {
            steps {
                echo '===== Setting up Python environment ====='

                sh '''
                    set -eu

                    python3.13 --version

                    rm -rf venv
                    python3.13 -m venv venv

                    . venv/bin/activate

                    python --version
                    python -m pip install --upgrade pip setuptools wheel
                    python -m pip install -r requirements.txt

                    echo "===== Checking installed dependencies ====="
                    python -m pip check
                '''
            }
        }

        stage('Validate Application') {
            steps {
                echo '===== Validating Python source ====='

                sh '''
                    set -eu
                    . venv/bin/activate

                    echo "===== Compile Python files ====="
                    python -m compileall -q .

                    echo "===== Import Flask application ====="
                    python -c "import main; print('Application import OK')"
                '''
            }
        }

        stage('Clean Deployment Artifacts') {
            steps {
                echo '===== Cleaning generated staging directory ====='

                sh '''
                    set -eu
                    if [ -d staging ]; then
                        rm -rf staging
                    fi
                '''
            }
        }

        stage('Authenticate and Verify GCP') {
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
                        gcloud auth list --filter=status:ACTIVE

                        echo "===== Verify App Engine ====="
                        gcloud app describe \
                            --project="$PROJECT_ID"
                    '''
                }
            }
        }

        stage('Deploy Without Promoting Traffic') {
            options {
                timeout(time: 25, unit: 'MINUTES')
            }

            steps {
                echo '===== Deploying candidate version ====='

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

                        gcloud app deploy app.yaml \
                            --project="$PROJECT_ID" \
                            --no-promote \
                            --quiet \
                            --verbosity=info

                        echo "===== Deployment command completed ====="
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Candidate deployment completed. Production traffic was not intentionally promoted.'
        }

        failure {
            echo 'Pipeline failed. Review the first failed stage and its logs.'
        }

        always {
            echo '===== Cleaning Jenkins workspace ====='
            cleanWs()
        }
    }
}
