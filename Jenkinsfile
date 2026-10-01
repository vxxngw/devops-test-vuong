def tg(String msg) {
  withCredentials([string(credentialsId: 'telegram-token', variable: 'TOKEN'),
                   string(credentialsId: 'telegram-chat-id', variable: 'CHAT')]) {
    sh """curl -s -X POST "https://api.telegram.org/bot\$TOKEN/sendMessage" \
          -d chat_id="\$CHAT" --data-urlencode "text=${msg}" > /dev/null"""
  }
}

pipeline {
  agent any
  triggers { pollSCM('H/1 * * * *') }   // tự kiểm tra GitHub mỗi phút
  environment {
    PROJECT  = 'devops-test-vuong'
    BRANCH   = 'main'
    APP      = 'devops-test-web'
    SITE_URL = 'http://localhost:8081'
  }

  stages {
    stage('Notify Start') {
      steps { tg("🚀 DEPLOY STARTED\nProject: ${PROJECT}\nBranch: ${BRANCH}") }
    }
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Install Dependencies') {
      steps {
        sh '''
          if [ -f package.json ]; then
            docker run --rm -u $(id -u):$(id -g) -e HOME=/tmp \
              -v "$WORKSPACE":/app -w /app node:20-alpine sh -c "npm ci || npm install"
          else
            echo "Static site - no dependencies"
          fi
        '''
      }
    }
    stage('Build') {
      steps {
        sh '''
          if [ -f package.json ] && grep -q '"build"' package.json; then
            docker run --rm -u $(id -u):$(id -g) -e HOME=/tmp \
              -v "$WORKSPACE":/app -w /app node:20-alpine npm run build
            if [ -d dist ]; then OUT=dist; else OUT=build; fi
          else
            OUT=.
          fi
          test -f "$OUT/index.html" || { echo "ERROR: index.html not found in $OUT"; exit 1; }

          echo "FROM nginx:alpine" > Dockerfile.web
          echo "COPY $OUT/ /usr/share/nginx/html/" >> Dockerfile.web
          echo ".git" > .dockerignore
          echo "node_modules" >> .dockerignore
          docker build -f Dockerfile.web -t $APP:latest .
        '''
      }
    }
    stage('Deploy') {
      steps {
        sh '''
          docker rm -f $APP || true
          docker run -d --name $APP -p 8081:80 $APP:latest
        '''
      }
    }
  }

  post {
    success { tg("✅ DEPLOY SUCCESS\nProject: ${PROJECT}\nBranch: ${BRANCH}\nURL: ${SITE_URL}") }
    failure { tg("❌ DEPLOY FAILED\nProject: ${PROJECT}\nBranch: ${BRANCH}\nPlease check Jenkins.") }
  }
}