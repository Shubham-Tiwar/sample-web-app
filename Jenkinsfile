pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Shubham-Tiwar/sample-web-app.git'
        APP_NAME = 'sample-web-app-1.0.jar'
        APP_LOG = '/tmp/app.log'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
        JOB_NAME = 'github-auto-build'
        DEPLOY_PORT = '8081'
        BUILD_DIR = "/var/lib/jenkins/workspace/github-auto-build"
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
                script {
                    // Save current successful build number
                    writeFile file: '/tmp/last_successful_build.txt', text: "${env.BUILD_NUMBER}"

                    sh """
                        set -e
                        cd ${BUILD_DIR}/target

                        if [ -f /tmp/springboot.pid ]; then
                            kill \$(cat /tmp/springboot.pid) || true
                            rm -f /tmp/springboot.pid
                        fi

                        echo "Starting Spring Boot app"
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
        }

         stage('Verify') {
            steps {
                sh """
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



    post {
        failure {
            echo '❌ Deployment failed. Rolling back to previous version...'

            script {
                def rollbackBuild = (env.BUILD_NUMBER.toInteger() - 1)
                if (rollbackBuild <= 0) {
                    echo "🚫 No previous build available for rollback."
                    return
                }

                sh """
                    set -e

                    if [ -f /tmp/springboot.pid ]; then
                        kill \$(cat /tmp/springboot.pid) || true
                        rm -f /tmp/springboot.pid
                    fi

                    mkdir -p ${BUILD_DIR}/target

                    echo "🔁 Rolling back to build #${rollbackBuild}"
                    cp ${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${rollbackBuild}/archive/target/${APP_NAME} ${BUILD_DIR}/target/${APP_NAME}

                    echo "▶️ Restarting old version"
                    cd ${BUILD_DIR}/target
                    JENKINS_NODE_COOKIE=dontKillMe nohup java -jar ${APP_NAME} --server.port=${DEPLOY_PORT} --server.address=0.0.0.0 > ${APP_LOG} 2>&1 & disown
                    echo \$! > /tmp/springboot.pid
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
}
}
