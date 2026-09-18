pipeline {
    agent any

    tools {
        maven 'maven'
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
                  -Dsonar.projectKey=olivialzx_springboot-aks-devops                  -Dsonar.projectName=springbootjavaapp \
                  -Dsonar.java.binaries=target/classes
            '''
        }
    }
}
    }
}