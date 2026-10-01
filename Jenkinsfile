pipeline {
    agent any

    environment {
        VERCEL_TOKEN = credentials('VERCEL_TOKEN')
        TELEGRAM_BOT_TOKEN = credentials('TELEGRAM_TOKEN')
        TELEGRAM_CHAT_ID = credentials('TELEGRAM_CHAT_ID')
        PROJECT_NAME = 'devops-test'
        BRANCH_NAME = 'main'
    }

    stages {
        stage('Notify Start') {
            steps {
                sh '''
                curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
                  -d chat_id=$TELEGRAM_CHAT_ID \
                  -d text="🚀 DEPLOY STARTED%0AProject: $PROJECT_NAME%0ABranch: $BRANCH_NAME"
                '''
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install dependencies') {
            steps {
                sh 'npm install -g vercel || true'
            }
        }
        stage('Build project') {
            steps {
                sh 'echo "Static site - validating files..."'
                sh 'test -f index.html && echo "index.html found"'
            }
        }
        stage('Deploy') {
            steps {
                sh 'vercel --token=$VERCEL_TOKEN --prod --yes > deploy_output.txt'
                sh 'cat deploy_output.txt'
            }
        }
    }

    post {
        success {
            script {
                def url = sh(script: "tail -n 1 deploy_output.txt", returnStdout: true).trim()
                sh """
                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                  -d chat_id=${TELEGRAM_CHAT_ID} \
                  -d text="✅ DEPLOY SUCCESS%0AProject: ${PROJECT_NAME}%0ABranch: ${BRANCH_NAME}%0AURL: ${url}"
                """
            }
        }
        failure {
            sh '''
            curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
              -d chat_id=$TELEGRAM_CHAT_ID \
              -d text="❌ DEPLOY FAILED%0AProject: $PROJECT_NAME%0ABranch: $BRANCH_NAME%0APlease check Jenkins."
            '''
        }
    }
}