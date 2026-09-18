
pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        ACR_SERVER = "oliviacontainerreg.azurecr.io"
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

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'acr-creds',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh '''
                        echo "$ACR_PASS" | docker login ${ACR_SERVER} \
                            -u "$ACR_USER" \
                            --password-stdin

                        docker push ${ACR_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout ${ACR_SERVER}
                    '''
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }
    }
}
