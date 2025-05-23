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
                    # Kiểm tra các công cụ cần thiết
                    echo "Kiểm tra các công cụ cần thiết..."
                    which git || echo "Git không được cài đặt"
                    which curl || echo "Curl không được cài đặt"
                    which jq || echo "jq không được cài đặt, nhưng sẽ tiếp tục"

                    # Kiểm tra và cài đặt Node.js phiên bản mới hơn nếu có thể
                    echo "Kiểm tra phiên bản Node.js..."
                    NODE_VERSION=$(node --version)
                    echo "Node.js version: $NODE_VERSION"

                    # Thử cài đặt NVM nếu chưa có
                    if [ ! -s "$HOME/.nvm/nvm.sh" ]; then
                        echo "NVM không được tìm thấy, thử cài đặt NVM..."
                        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash || echo "Không thể cài đặt NVM"

                        # Kiểm tra lại NVM
                        if [ -s "$HOME/.nvm/nvm.sh" ]; then
                            echo "NVM đã được cài đặt thành công"
                        else
                            echo "Không thể cài đặt NVM, tiếp tục với Node.js hiện tại"
                        fi
                    else
                        echo "NVM đã được cài đặt"
                    fi

                    # Thử sử dụng NVM để cài đặt Node.js 20
                    if [ -s "$HOME/.nvm/nvm.sh" ]; then
                        echo "Thử cài đặt Node.js 20 với NVM..."
                        export NVM_DIR="$HOME/.nvm"
                        . "$NVM_DIR/nvm.sh"
                        nvm install 20 || echo "Không thể cài đặt Node.js 20 với NVM"
                        nvm use 20 || echo "Không thể sử dụng Node.js 20"
                        NODE_VERSION=$(node --version)
                        echo "Node.js version sau khi cài đặt: $NODE_VERSION"
                    else
                        echo "Thử cài đặt Node.js 20 với n..."
                        # Thử cài đặt n và sử dụng nó để cài đặt Node.js 20
                        npm install -g n || echo "Không thể cài đặt n"
                        n 20 || echo "Không thể cài đặt Node.js 20 với n"
                        NODE_VERSION=$(node --version)
                        echo "Node.js version sau khi cài đặt: $NODE_VERSION"
                    fi

                    # Cài đặt Firebase CLI cục bộ
                    echo "Installing Firebase CLI..."
                    # Tạo package.json nếu chưa tồn tại
                    if [ ! -f "package.json" ]; then
                        echo '{"name":"firebase-deploy","private":true}' > package.json
                    fi
                    # Cài đặt Firebase CLI cục bộ với cờ --force để bỏ qua cảnh báo về phiên bản Node.js
                    npm install firebase-tools --no-fund --no-audit --no-package-lock --force

                    # Thiết lập biến môi trường cho Firebase credentials
                    export GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS}

                    # Hiển thị thông tin về file credentials
                    echo "Checking credentials file..."
                    ls -la ${GOOGLE_APPLICATION_CREDENTIALS}

                    # Đảm bảo file credentials có quyền đọc
                    echo "Đảm bảo file credentials có quyền đọc..."
                    chmod 644 ${GOOGLE_APPLICATION_CREDENTIALS} || echo "Không thể thay đổi quyền truy cập của file credentials"

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
                    ./node_modules/.bin/firebase deploy --only hosting --project=${FIREBASE_PROJECT_ID} --non-interactive --force
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
