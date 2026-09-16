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

    }
}