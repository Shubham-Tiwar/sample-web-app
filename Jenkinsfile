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
                   cd /var/lib/jenkins/workspace/github-auto-build/target
                   if [ -f /tmp/springboot.pid ]; then
                   kill $(cat /tmp/springboot.pid) || true
                   rm -f /tmp/springboot.pid
                   fi

                   echo "Starting Spring Boot app in background"
                   nohup java -jar sample-web-app-1.0.jar \
                   --server.port=8081 \
                   --server.address=0.0.0.0 > ~/app.log 2>&1 &

                   echo $! > /tmp/springboot.pid
                   sleep 15

                   echo "--- Netstat Check ---"
                   ss -tuln | grep 8081 || echo "⚠️ Port 8081 not active"

                   echo "--- Tail Log ---"
                   tail -n 20 ~/app.log
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

