pipeline {
    agent {
        docker {
            image 'node:20-alpine'
            args '-u root'
        }
    }

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

                    # Kiểm tra nội dung file cấu hình
                    echo "Kiểm tra nội dung file cấu hình Firebase..."
                    cat firebase.json
                    cat .firebaserc

                    # Kiểm tra file credentials
                    if [ ! -f "${GOOGLE_APPLICATION_CREDENTIALS}" ]; then
                        echo "CẢNH BÁO: Không tìm thấy file credentials Firebase tại ${GOOGLE_APPLICATION_CREDENTIALS}"
                        echo "Vui lòng kiểm tra cấu hình credentials trong Jenkins"
                    else
                        echo "File credentials Firebase tồn tại"
                    fi

                    echo "Cấu trúc dự án hợp lệ"
                '''
            }
        }

        stage('Deploy') {
            steps {
                // Deploy lên Firebase Hosting
                sh '''
                    # Cài đặt các công cụ cần thiết
                    echo "Cài đặt các công cụ cần thiết..."
                    apk add --no-cache git curl jq

                    # Hiển thị phiên bản Node.js
                    echo "Node.js version:"
                    node --version

                    # Cài đặt Firebase CLI cục bộ
                    echo "Installing Firebase CLI..."
                    # Tạo package.json nếu chưa tồn tại
                    if [ ! -f "package.json" ]; then
                        echo '{"name":"firebase-deploy","private":true}' > package.json
                    fi
                    # Cài đặt Firebase CLI cục bộ
                    npm install firebase-tools --no-fund --no-audit --no-package-lock

                    # Thiết lập biến môi trường cho Firebase credentials
                    export GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS}

                    # Hiển thị thông tin về file credentials
                    echo "Checking credentials file..."
                    ls -la ${GOOGLE_APPLICATION_CREDENTIALS}

                    # Kiểm tra jq đã được cài đặt chưa
                    echo "Kiểm tra jq..."
                    which jq

                    # Sử dụng Firebase CLI cục bộ
                    echo "Deploying to Firebase..."

                    # Hiển thị phiên bản Node.js và Firebase CLI
                    echo "Node.js version:"
                    node --version
                    echo "Firebase CLI version:"
                    ./node_modules/.bin/firebase --version

                    # Thiết lập biến môi trường cho Firebase credentials
                    export GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS}

                    # Hiển thị cấu trúc file credentials (che giấu giá trị thực)
                    echo "Cấu trúc file credentials:"
                    jq 'keys' ${GOOGLE_APPLICATION_CREDENTIALS}

                    # Triển khai lên Firebase Hosting sử dụng service account
                    echo "Triển khai lên Firebase Hosting..."

                    # Sử dụng biến môi trường GOOGLE_APPLICATION_CREDENTIALS
                    echo "Sử dụng service account từ biến môi trường GOOGLE_APPLICATION_CREDENTIALS"
                    ./node_modules/.bin/firebase use ${FIREBASE_PROJECT_ID}
                    ./node_modules/.bin/firebase deploy --only hosting --project=${FIREBASE_PROJECT_ID} --non-interactive
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

            // Thêm thông báo Slack nếu đã cấu hình
            script {
                try {
                    slackSend(
                        color: 'good',
                        message: "Deployment SUCCESSFUL: Website đã được deploy tại https://${FIREBASE_PROJECT_ID}.web.app"
                    )
                } catch (Exception e) {
                    echo "Không thể gửi thông báo Slack: ${e.message}"
                }
            }
        }
        failure {
            // Gửi thông báo khi deploy thất bại
            echo 'Deployment failed!'

            // Thêm thông báo Slack nếu đã cấu hình
            script {
                try {
                    slackSend(
                        color: 'danger',
                        message: "Deployment FAILED: Kiểm tra Jenkins logs để biết thêm chi tiết"
                    )
                } catch (Exception e) {
                    echo "Không thể gửi thông báo Slack: ${e.message}"
                }
            }
        }
        always {
            // Dọn dẹp workspace sau khi hoàn thành
            cleanWs()
        }
    }
}
