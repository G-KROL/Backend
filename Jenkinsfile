def imageName="192.168.44.44:8082/docker_registry/backend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"
def dockerTag=""

pipeline {
    agent {
        label 'agent'
    }
    
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }    
    }
    
    environment {
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }

        stage('Unit tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }

        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build application image') {
            steps {
                script {
                  dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
                  //dockerTag = "RC-${env.BUILD_ID}"
                  applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
        stage('Push image to Artifactory') {
            steps {
                script {
                  docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                      applicationImage.push()
                      applicationImage.push('latest')
                  }
                }
            }
        }
    }
}