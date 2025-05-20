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
                        sh """
                            export SONAR_TOKEN=${SONAR_TOKEN}
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                echo "🔄 Killing existing app on port 8081 if any..."
                fuser -k 8081/tcp || true

                echo "🚀 Deploying the app..."
                cd /var/lib/jenkins/workspace/github-auto-build/target
                nohup java -jar ${APP_NAME} --server.port=8081 --server.address=0.0.0.0 > ${APP_LOG} 2>&1 &

                echo "⏳ Waiting for app to start..."
                for i in {1..15}; do
                    curl -s http://127.0.0.1:8081 && echo "✅ App is up!" && break
                    sleep 2
                done
                """
            }
        }

        stage('Verify') {
            steps {
                sh """
                echo "🔍 Verifying if app is accessible..."

                if curl -s http://127.0.0.1:8081 > /dev/null; then
                    echo "✅ App is reachable"
                else
                    echo "❌ App is NOT reachable"
                    echo "--- Last 30 lines of App Log ---"
                    tail -n 30 ${APP_LOG}
                    exit 1
                fi
                """
            }
        }
    }
}

