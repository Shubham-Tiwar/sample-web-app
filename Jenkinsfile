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

        APP_JAR=target/${APP_NAME}
        LOG_FILE=/tmp/app.log
        PID_FILE=/tmp/springboot.pid

        # Stop previous app if running
        if [ -f $PID_FILE ]; then
            echo "Stopping old app..."
            kill $(cat $PID_FILE) || true
            rm -f $PID_FILE
        fi

        echo "Starting app..."
        nohup java -jar $APP_JAR --server.port=8081 --server.address=0.0.0.0 > $LOG_FILE 2>&1 &

        echo $! > $PID_FILE
        sleep 10

        echo "--- Port Check ---"
        ss -tuln | grep 8081 || echo "⚠️ Port 8081 not active"

        echo "--- Log Preview ---"
        tail -n 20 $LOG_FILE
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

