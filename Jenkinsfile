pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        ACR_SERVER = "democontainerreg.azurecr.io"
        IMAGE_NAME = "springbootjavaapp"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout from Git') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/olivialzx/springboot-aks-devops.git'
            }
        }

        stage('Maven Validate') {
            steps {
                sh 'mvn validate'
            }
        }

        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.organization=bootcamp2 \
                          -Dsonar.projectKey=olivialzx_springboot-aks-devops \
                          -Dsonar.projectName=springbootjavaapp \
                          -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Package with Maven') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${ACR_SERVER}/${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }
    }
}