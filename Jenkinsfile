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
        stage ('Install Dependies') {
            steps {
                script {
                    sh """
                     npm install 

                     """
                
                }
            
            
            }
        }
        stage ('unit testing') {
            steps {
                script {
                sh """
                echo "unit testing "
                """

                }
            }

        }
        stage ('Docker build'){
            steps {
                script {
                    withAWS(credentials: 'aws-auth', region: 'us-east-1') {
                    sh """ 
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com
                        
                        docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion} .
                        docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion}

                    """
                    
                    
                    }
                }
        }    }
    
    
    }



}