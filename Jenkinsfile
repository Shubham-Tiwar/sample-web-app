pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Shubham-Tiwar/sample-web-app.git'
        APP_NAME = 'sample-web-app-1.0.jar'
        APP_LOG = '/tmp/app.log'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
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

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            export SONAR_TOKEN=${SONAR_TOKEN}
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                set -e

                JAR_PATH=target/${APP_NAME}

                if [ ! -f "$JAR_PATH" ]; then
                    echo "❌ JAR file not found at $JAR_PATH"
                    exit 1
                fi

                echo "🔁 Killing previous app if running"
                if [ -f /tmp/springboot.pid ]; then
                    kill $(cat /tmp/springboot.pid) || true
                    rm -f /tmp/springboot.pid
                fi

                echo "🚀 Starting Spring Boot app"
                nohup java -jar $JAR_PATH \
                    --server.port=8081 \
                    --server.address=0.0.0.0 > ${APP_LOG} 2>&1 &

                echo $! > /tmp/springboot.pid
                sleep 5

                echo "⏳ Waiting for app to start on port 8081"
                for i in {1..10}; do
                    if curl -s http://127.0.0.1:8081 > /dev/null; then
                        echo "✅ App is running"
                        break
                    else
                        echo "⏱️ Not ready yet..."
                        sleep 2
                    fi
                done

                echo "📡 Netstat check:"
                ss -tuln | grep 8081 || echo "⚠️ Port 8081 not active"

                echo "📜 Last log lines:"
                tail -n 20 ${APP_LOG}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                echo "🔍 Verifying app"
                if curl -s http://127.0.0.1:8081 > /dev/null; then
                    echo "✅ App is reachable"
                else
                    echo "❌ App is not reachable"
                    echo "--- LOG ---"
                    tail -n 30 ${APP_LOG}
                    exit 1
                fi
                '''
            }
        }
    }
}

