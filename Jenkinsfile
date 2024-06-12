pipeline {
    agent any
    
    tools {
        maven 'Mvn'
    }

    environment {
        DOCKER_IMAGE = 'satishmoyya/spc'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/satishmoyya/spring-petclinic.git'
            }
        }
        stage('mvn build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('sonarqube') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh '''
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=spc \
                    -Dsonar.projectName='spc' \
                    -Dsonar.host.url=http://34.125.145.8:9000 \
                    -Dsonar.token=squ_ef5d4cb7f5a3b8ef4c273851b7067276b858b9b8
                    '''
                }
            }
        }
        stage('docker build and push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerhub',
                                                  usernameVariable: 'DOCKER_USERNAME',
                                                  passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        // Build Docker image
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                        // Login to Docker Hub
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        // Push Docker image
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend color: '#36A64F', message: "Pipeline success - ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: '#FF0000', message: "Pipeline failed! - ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
        }
    }
}
