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

parameters{
booleanParam(name: 'deploy', defaultValue: false, description: 'toogle this value')
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
                    echo "Running unit tests"
                '''
            }
        }
    }

    stage('Docker Build & Push') {
        steps {
            script {
                withAWS(credentials: 'aws-auth', region: "${region}") {
                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com
                        
                        export DOCKER_BUILDKIT=0
                        
                        docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion} .
                        
                        docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion}
                    """
                }
            }
        }
    }

    stage('Trigger Deploy') {
    when {
        expression {params.deploy}
    }
        steps {
            script {
                build job: 'catalogue-cd',
                parameters:[
                string(name: 'appVersion', value:"${appVersion}")
                string(name: 'deploy_to', value:'dev')
                ],
                propagate: false,  // even SG fails VPC will not be effected
                wait: false
            }
        }
    }

}


}
