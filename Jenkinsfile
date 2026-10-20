
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install --upgrade pip
                    .venv/bin/pip install -r requirements.txt
                    .venv/bin/pip install -r requirements-dev.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '.venv/bin/python -m pytest -q'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t drug-tariff-analytics:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy to Render') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'render-deploy-hook',
                        variable: 'RENDER_DEPLOY_HOOK'
                    )
                ]) {
                    sh '''
                        curl --fail --silent --show-error \
                            -X POST "$RENDER_DEPLOY_HOOK"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Tests, Docker build, and Render deployment trigger completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}
