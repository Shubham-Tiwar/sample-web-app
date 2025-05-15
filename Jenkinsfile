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
                set -e
                cd target

                if [ -f /tmp/springboot.pid ]; then
                    kill $(cat /tmp/springboot.pid) || true
                    rm -f /tmp/springboot.pid
                fi

                echo "Starting Spring Boot app in background"
                nohup java -jar ${APP_NAME} \
                    --server.port=8081 \
                    --server.address=0.0.0.0 > ${APP_LOG} 2>&1 &

                echo $! > /tmp/springboot.pid
                sleep 10

                echo "--- Netstat Check ---"
                netstat -tuln | grep 8081 || echo "⚠️ Port 8081 not active"

                echo "--- Tail Log ---"
                tail -n 20 ${APP_LOG}
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
