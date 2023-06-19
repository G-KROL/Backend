def imageName="bruno11n1/backend"
def dockerRegistry=""
def registryCredentials="5d465160-176b-461f-b0eb-86c49ff5d374"
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

   /*      stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        } */

        stage('Build application image') {
            steps {
                script {
                  dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
                  //dockerTag = "RC-${env.BUILD_ID}"
                  applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
              stage ('Push to repo') {
            steps {
                dir('ArgoCD') {
                    withCredentials([gitUsernamePassword(credentialsId: '819e4d89-9fb1-49ac-9d4b-a548a2978441', gitToolName: 'Default')]) {
                        git branch: 'master', url: 'https://github.com/G-KROL/ArgoCD'
                        sh """ cd backend
                        git config --global user.email "grzegorzkrol90@gmail.com"
                        git config --global user.name "gkrol"
                        sed -i "s#$imageName.*#$imageName:$dockerTag#g" backend_deployment.yaml
                        git commit -am "Set new $dockerTag tag."
                        git diff
                        git push origin master
                        """
                    }                  
                } 
            }
        }
      /*   stage('Push image to Artifactory') {
            steps {
                script {
                  docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                      applicationImage.push()
                      applicationImage.push('latest')
                  }
                }
            }
        } */
    }
}