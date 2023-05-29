def imageName="192.168.44.44:8082/docker_registry/backend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"
def dockerTag=""

pipeline {
    agent {
      label 'agent'
    }
    environment {
      scannerHome = tool 'SonarQube'
    }
    stages {
      stage('checkout repo') {
        steps {
          //git branch: 'tests', url: 'https://github.com/gajdamariusz84/Backend.git'
          checkout scm
        }
      }
      stage('test') {
        steps {
          sh 'ls -la'
          sh 'pip3 install -r requirements.txt'
          sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
        }
      }
    stage('run SonarQube') {
        steps {
            withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
            timeout(1) {
                waitForQualityGate false
            }
          }
        }
    stage('build app image') {
        steps {
            script {
                dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
                applicationImage = docker.build("$imageName:$dockerTag",".")
            }
          }    
        }
    stage('push image to artifactory') {
        steps {
            script {
                docker.withRegistry("$dockerRegistry","$registryCredentials") {
                    applicationImage.push()
                    applicationImage.push('latest')
                }
            }    
          }
        }
    }
    post {
    	always {
    		junit testResults: "test-results/*.xml"
    		cleanWs()
    	}
    }

}