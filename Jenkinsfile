pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Thakur156/Python-Webapp'
            }
        }
        
         stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy Fs Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('SONARQUBE ANALYSIS') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=p-webapp \
                        -Dsonar.projectKey=p-webapp
                    """
                }
            }
        }
        
        stage('Docker build & tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh "make image"
                    }
                } 
            }
        }
        
        stage('Trivy image Scan') {
            steps {
                sh "trivy image thakur156/python-webapp:latest "
            }
        }
        
        stage('Docker Push') {
            steps {
                sh "make push "
            }
        }
        
        stage('Deploy to container') {
            steps {
                sh "docker run -d -p 5000:5000 thakur156/python-webapp:latest "
            }
        }
    }
}
