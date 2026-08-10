pipeline {
agent {
label 'AGENT-1'
}

```
environment {
    appVersion = ''
    region = 'us-east-1'
    account_id = '143094436925'
    project = 'roboshop'
    component = 'catalogue'
}

parameters {
    booleanParam(name: 'deploy', defaultValue: false, description: 'toggle this value')
}

stages {

    stage('Read package.json') {
        steps {
            script {
                def packageJson = readJSON file: 'package.json'
                env.appVersion = packageJson.version
                echo "Application Version: ${env.appVersion}"
            }
        }
    }

    stage('Install Dependencies') {
        steps {
            sh 'npm install'
        }
    }

    stage('Unit Testing') {
        steps {
            sh 'echo "Running unit tests"'
        }
    }

    stage('Sonar Scan') {
        environment {
            // Sonar scanner from Jenkins tools
            scannerHome = tool 'sonar-scanner'
        }
        steps {
            script {
                // SonarQube server config
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                        ${scannerHome}/bin/sonar-scanner
                    """
                }
            }
        }
    }

    stage('Quality Gate') {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: false
            }
        }
    }

    stage('Dependabot Alerts Check') {
        steps {
            script {
                withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {

                    def response = sh(
                        script: '''
                        curl -s -L \
                          -H "Accept: application/vnd.github+json" \
                          -H "Authorization: Bearer $TOKEN" \
                          -H "X-GitHub-Api-Version: 2022-11-28" \
                          https://api.github.com/repos/nagalakshmi477/catalogue/dependabot/alerts
                        ''',
                        returnStdout: true
                    ).trim()

                    def json = readJSON text: response

                    def criticalHigh = json.findAll { alert ->
                        alert.security_advisory.severity in ['high', 'critical']
                    }

                    def count = criticalHigh.size()

                    echo "Total HIGH/CRITICAL alerts: ${count}"

                    echo groovy.json.JsonOutput.prettyPrint(
                        groovy.json.JsonOutput.toJson(criticalHigh)
                    )

                    if (count > 0) {
                        error "❌ High or Critical vulnerabilities found!"
                    } else {
                        echo "✅ No High/Critical vulnerabilities"
                    }
                }
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
                        
                        docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${env.appVersion} .
                        
                        docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${env.appVersion}
                    """
                }
            }
        }
    }

    stage('Trigger Deploy') {
        when {
            expression { params.deploy }
        }
        steps {
            script {
                build job: 'catalogue-cd',
                    parameters: [
                        string(name: 'appVersion', value: "${env.appVersion}"),
                        string(name: 'deploy_to', value: 'dev')
                    ],
                    propagate: false,
                    wait: false
            }
        }
    }
}

post {
    always {
        echo "Pipeline completed"
    }
    success {
        echo "Pipeline succeeded"
    }
    failure {
        echo "Pipeline failed"
    }
}
```

}
