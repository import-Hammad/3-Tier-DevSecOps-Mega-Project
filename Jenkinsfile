pipeline {
    agent any
    tools {
        nodejs 'nodejs23'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'docker-build-deploy', url: 'https://github.com/import-Hammad/3-Tier-DevSecOps-Mega-Project.git'
            }
        }
        stage('Frontend compilation') {
            steps {
                dir('client'){
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        stage('backend compilation') {
            steps {
                dir('api'){
                    sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        stage('gitleaks scan') {
            steps {
                sh 'gitleaks detect --source ./client --exit-code 1'
                sh 'gitleaks detect --source ./api --exit-code 1'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=NodeJS-Project \
                            -Dsonar.projectKey=NodeJS-Project '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Trivy Fs scan') {
            steps {
                sh 'trivy fs --timeout 30m --format table -o fs-report.html .'
            }
        }
        stage('Build and tag docker image for backend') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Dockerhub_credentials') {
                        dir('api') {
                            sh 'docker build -t piratehammad/backend:latest .'
                            sh 'trivy image --format table -o backend-image-report.html piratehammad/backend:latest'
                            sh 'docker push piratehammad/backend:latest'
                        }
                    }
                }
            }
        }
        stage('Build and tag docker image for frontend') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Dockerhub_credentials') {
                        dir('client') {
                            sh 'docker build -t piratehammad/frontend:latest .'
                            sh 'trivy image --format table -o frontend-image-report.html piratehammad/frontend:latest'
                            sh 'docker push piratehammad/frontend:latest'
                        }
                    }
                }
            }
        }
        stage('docker deploy via compose') {
            steps {
                script {
                    sh 'docker compose up -d'
                }
            }
        }
    }
}
