pipeline {
    agent any

    environment {
        // Project ID từ file .firebaserc
        FIREBASE_PROJECT_ID = 'hungdt-jenkin'
        // Đường dẫn đến file credentials Firebase
        GOOGLE_APPLICATION_CREDENTIALS = credentials('firebase-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                // Lấy mã nguồn từ repository
                checkout scm
                echo 'Checkout completed'
            }
        }

        stage('Verify') {
            steps {
                // Kiểm tra cấu trúc dự án
                sh '''
                    echo "Kiểm tra cấu trúc dự án..."
                    ls -la

                    # Kiểm tra file cấu hình Firebase
                    if [ ! -f "firebase.json" ]; then
                        echo "Không tìm thấy file firebase.json"
                        exit 1
                    fi

                    if [ ! -f ".firebaserc" ]; then
                        echo "Không tìm thấy file .firebaserc"
                        exit 1
                    fi

                    echo "Cấu trúc dự án hợp lệ"
                '''
            }
        }

        stage('Deploy') {
            steps {
                // Deploy lên Firebase Hosting
                sh '''
                    # Cài đặt Firebase CLI nếu chưa có
                    if ! command -v firebase &> /dev/null; then
                        echo "Installing Firebase CLI..."
                        npm install -g firebase-tools
                    fi

                    # Thiết lập biến môi trường cho Firebase credentials
                    export GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS}

                    # Hiển thị thông tin về file credentials
                    ls -la ${GOOGLE_APPLICATION_CREDENTIALS}

                    # Deploy lên Firebase Hosting
                    firebase deploy --only hosting --project=${FIREBASE_PROJECT_ID}
                '''
                echo 'Deployment completed'
            }
        }
    }

    post {
        success {
            // Gửi thông báo khi deploy thành công
            echo 'Deployment successful!'

            // Gửi thông báo qua Slack
            slackSend(
                color: 'good',
                message: "🚀 Deploy thành công: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nProject: ${FIREBASE_PROJECT_ID}\nURL: https://${FIREBASE_PROJECT_ID}.web.app\n(<${env.BUILD_URL}|Chi tiết>)"
            )

            // Gửi email thông báo (tùy chọn)
            emailext (
                subject: "✅ SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>
                <p>Website đã được deploy tại: <a href='https://${FIREBASE_PROJECT_ID}.web.app'>https://${FIREBASE_PROJECT_ID}.web.app</a></p>""",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
        failure {
            // Gửi thông báo khi deploy thất bại
            echo 'Deployment failed!'

            // Gửi thông báo qua Slack
            slackSend(
                color: 'danger',
                message: "❌ Deploy thất bại: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nProject: ${FIREBASE_PROJECT_ID}\n(<${env.BUILD_URL}|Xem log lỗi>)"
            )

            // Gửi email thông báo lỗi (tùy chọn)
            emailext (
                subject: "❌ FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
        always {
            // Dọn dẹp workspace sau khi hoàn thành
            cleanWs()
        }
    }
}
