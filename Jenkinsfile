pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Shubham-Tiwar/sample-web-app.git'
        APP_NAME = 'sample-web-app-1.0.jar'
        APP_LOG = '/tmp/app.log'
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
                sh '''
                cd target
                fuser -k 8081/tcp || true
                nohup java -jar ${APP_NAME} > ${APP_LOG} 2>&1 &
                sleep 5
                netstat -tuln | grep 8081 || true
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'curl -s http://localhost:8081 || echo "⚠️ App not reachable"'
                sh 'echo "--- Last 20 lines of log ---"'
                sh 'tail -n 20 ${APP_LOG} || echo "⚠️ No log found"'
            }
        }
    }
}
