pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Shubham-Tiwar/sample-web-app.git'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                dir('target') {
                    sh 'fuser -k 8081/tcp || true'
                    sh 'nohup java -jar sample-web-app-1.0.jar > ../app.log 2>&1 &'
                    sh 'sleep 5'
                    sh 'ss -tuln | grep 8081 || true'
                }
            }
        }
    }
}
