pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Shubham-Tiwar/sample-web-app.git'
        APP_NAME = 'sample-web-app-1.0.jar'
        APP_LOG = '/tmp/app.log'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
        JOB_NAME = 'github-auto-build'
        DEPLOY_PORT = '8082'
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
                archiveArtifacts artifacts: "target/${APP_NAME}", fingerprint: true
                sh 'echo ${BUILD_NUMBER} > /tmp/last_successful_build.txt'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    set -e
                    cd /var/lib/jenkins/workspace/${JOB_NAME}/target

                    # Stop any previous app if running
                    if [ -f /tmp/springboot.pid ]; then
                        kill \$(cat /tmp/springboot.pid) || true
                        rm -f /tmp/springboot.pid
                    fi

                    echo "Starting Spring Boot app in background"
                    JENKINS_NODE_COOKIE=dontKillMe nohup java -jar ${APP_NAME} --server.port=${DEPLOY_PORT} --server.address=0.0.0.0 > ${APP_LOG} 2>&1 & disown
                    echo \$! > /tmp/springboot.pid
                    sleep 10

                    echo "--- Netstat Check ---"
                    ss -tuln | grep ${DEPLOY_PORT} || echo "⚠️ Port ${DEPLOY_PORT} not active"

                    echo "--- Tail Log ---"
                    tail -n 20 ${APP_LOG}
                """
            }
        }

        stage('Verify') {
            steps {
                sh """
                    #!/bin/bash
                    echo "🔍 Verifying if app is accessible..."

                    if curl -s http://127.0.0.1:${DEPLOY_PORT} > /dev/null; then
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

    post {
        failure {
            echo '❌ Deployment failed. Rolling back to previous version...'
            sh """
                #!/bin/bash
                set -e
                PREV_BUILD=$(expr $(cat /tmp/last_successful_build.txt) - 1)
                if [ $PREV_BUILD -le 0 ]; then
                    echo "No previous build available for rollback."
                    exit 1
                fi

                #cd /var/lib/jenkins/workspace/${JOB_NAME}
                #mkdir -p target

                if [ -f /tmp/springboot.pid ]; then
                    kill $(cat /tmp/springboot.pid) || true
                    rm -f /tmp/springboot.pid
                fi

                echo "Rolling back to build #$PREV_BUILD"
                cp /var/lib/jenkins/jobs/${JOB_NAME}/builds/$PREV_BUILD/archive/target/${APP_NAME} target/${APP_NAME}
                JENKINS_NODE_COOKIE=dontKillMe nohup java -jar target/${APP_NAME} --server.port=${DEPLOY_PORT} --server.address=0.0.0.0 > ${APP_LOG} 2>&1 & disown
               # nohup java -jar target/${APP_NAME} --server.port=${DEPLOY_PORT} --server.address=0.0.0.0 > ${APP_LOG} 2>&1 & disown
                echo $! > /tmp/springboot.pid
                sleep 10

                echo "--- Rollback Verification ---"
                if curl -s http://127.0.0.1:${DEPLOY_PORT} > /dev/null; then
                    echo "✅ Rollback successful"
                else
                    echo "❌ Rollback failed"
                    exit 1
                fi
            """
        }
    }
}

