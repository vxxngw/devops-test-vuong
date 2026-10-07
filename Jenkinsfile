def tg(String msg) {
  withCredentials([string(credentialsId: 'telegram-token', variable: 'TOKEN'),
                   string(credentialsId: 'telegram-chat-id', variable: 'CHAT')]) {
    sh """curl -s -X POST "https://api.telegram.org/bot\$TOKEN/sendMessage" \
          -d chat_id="\$CHAT" --data-urlencode "text=${msg}" > /dev/null"""
  }
}

pipeline {
  agent any
  triggers { pollSCM('H/1 * * * *') }   // Tự động kiểm tra GitHub mỗi phút

  environment {
    REPO_NAME   = 'devops-test-vuong'
    BRANCH      = 'main'
    SITE_URL    = 'https://devops-test-vuong.vercel.app'
    VERCEL_HOOK = 'https://api.vercel.com/v1/integrations/deploy/prj_AKrJIziA5G1E5qfXT0AM6rtiJxFc/my8LDAEnbU'
    COMMIT_HASH = 'unknown'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        script {
          env.COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
      }
    }

    stage('Notify Start') {
      steps {
        tg("🚀 Bắt đầu deploy website\nRepository: ${REPO_NAME}\nBranch: ${BRANCH}\nCommit: ${COMMIT_HASH}")
      }
    }

    stage('Build & Test') {
      steps {
        sh '''
          echo "Kiểm tra mã nguồn..."
          test -f index.html || { echo "ERROR: index.html không tồn tại"; exit 1; }
          echo "Mã nguồn hợp lệ."
        '''
      }
    }

    stage('Deploy to Vercel') {
      steps {
        sh '''
          echo "Gửi trigger deploy tới Vercel..."
          RESPONSE=$(curl -s -X POST "$VERCEL_HOOK")
          echo "Vercel Response: $RESPONSE"

          if echo "$RESPONSE" | grep -q '"job"'; then
            echo "Vercel deploy đã được kích hoạt thành công!"
          else
            echo "Lỗi khi kích hoạt Vercel deploy: $RESPONSE"
            exit 1
          fi
        '''
      }
    }
  }

  post {
    success {
      tg("✅ Deploy thành công\nRepository: ${REPO_NAME}\nBranch: ${BRANCH}\nWebsite: ${SITE_URL}")
    }
    failure {
      tg("❌ Deploy thất bại\nRepository: ${REPO_NAME}\nBranch: ${BRANCH}\nCommit: ${COMMIT_HASH}\nError: Jenkins pipeline execution failed")
    }
  }
}