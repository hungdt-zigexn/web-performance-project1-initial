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
                    # Kiểm tra Firebase CLI
                    echo "Checking Firebase CLI..."
                    which firebase || echo "Firebase CLI not found"

                    # Thiết lập biến môi trường cho Firebase credentials
                    export GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS}

                    # Hiển thị thông tin về file credentials
                    ls -la ${GOOGLE_APPLICATION_CREDENTIALS}

                    # Deploy lên Firebase Hosting
                    echo "Deploying to Firebase..."
                    firebase deploy --only hosting --project=${FIREBASE_PROJECT_ID} --non-interactive
                '''
                echo 'Deployment completed'
            }
        }
    }

    post {
        success {
            // Gửi thông báo khi deploy thành công
            echo 'Deployment successful!'
            echo "Website đã được deploy tại: https://${FIREBASE_PROJECT_ID}.web.app"
        }
        failure {
            // Gửi thông báo khi deploy thất bại
            echo 'Deployment failed!'
        }
        always {
            // Dọn dẹp workspace sau khi hoàn thành
            cleanWs()
        }
    }
}
