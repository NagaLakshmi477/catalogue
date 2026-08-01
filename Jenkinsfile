pipeline {
    agent {
        label 'AGENT-1'
    }

    environment {
        appVersion = ''
        region = 'us-east-1'
        account_id = '143094436925'
        project = 'roboshop'
        component = 'catalogue'
    }

    stages {

        stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Application Version: ${appVersion}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        npm install
                    '''
                }
            }
        }

        stage('Unit Testing') {
            steps {
                script {
                    sh '''
                        echo "unit testing"
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withAWS(credentials: 'aws-auth', region: "${region}") {
                        sh """
                            echo "Verifying AWS Credentials..."
                            aws sts get-caller-identity

                            echo "Logging into ECR..."
                            aws ecr get-login-password --region ${region} | \
                            docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                            echo "Building Docker Image..."
                            docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion} .

                            echo "Pushing Docker Image..."
                            docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion}
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed. Check AWS credentials or Docker login."
        }
        success {
            echo "Pipeline executed successfully."
        }
    }
}