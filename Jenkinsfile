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

                  # Stop existing app
                  if [ -f /tmp/springboot.pid ]; then
                  echo "Stopping existing app..."
                  kill $(cat /tmp/springboot.pid) || true
                  rm -f /tmp/springboot.pid
                  fi

                  echo "Starting Spring Boot app in background..."
                  nohup java -jar target/sample-web-app-1.0.jar \
                  --server.port=8081 \
                  --server.address=0.0.0.0 > /tmp/app.log 2>&1 &

                  echo $! > /tmp/springboot.pid
                  sleep 10

                  echo "Checking if app is running on port 8081..."
                  ss -tunlp | grep 8081 || (echo "❌ App not running on 8081" && tail -n 50 /tmp/app.log && exit 1)
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

