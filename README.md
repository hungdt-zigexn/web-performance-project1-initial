# Tự động deploy frontend app lên Firebase bằng Jenkins CI/CD

Dự án này sử dụng Jenkins để tự động deploy trang landing page lên Firebase Hosting mỗi khi có commit mới vào nhánh `main`.

## Cấu trúc dự án

- `index.html`: Trang landing page chính
- `css/`: Thư mục chứa các file CSS
- `js/`: Thư mục chứa các file JavaScript
- `images/`: Thư mục chứa hình ảnh
- `firebase.json`: Cấu hình Firebase Hosting
- `.firebaserc`: Cấu hình project Firebase
- `Jenkinsfile`: Cấu hình Jenkins pipeline

## Thiết lập Jenkins

### 1. Cài đặt Jenkins

Nếu bạn chưa cài đặt Jenkins, hãy tham khảo [hướng dẫn cài đặt Jenkins](https://www.jenkins.io/doc/book/installing/).

### 2. Cài đặt các plugin cần thiết

- Pipeline
- Git
- Slack Notification
- Email Extension

### 3. Thiết lập Credentials trong Jenkins

Truy cập Jenkins > Manage Jenkins > Manage Credentials > System > Global credentials > Add Credentials

#### Firebase Credentials

1. Tạo file credentials Firebase từ Google Cloud Console
2. Thêm credentials với ID là `firebase-credentials` và loại là "Secret file"
3. Upload file credentials Firebase (JSON)

#### Slack Webhook (nếu sử dụng Slack)

1. Tạo Webhook URL từ Slack
2. Thêm credentials với ID là `slack-webhook` và loại là "Secret text"
3. Nhập Webhook URL vào trường Secret

### 4. Thiết lập Webhook GitHub/GitLab

#### GitHub

1. Truy cập repository > Settings > Webhooks > Add webhook
2. Payload URL: `http://your-jenkins-url/github-webhook/`
3. Content type: `application/json`
4. Chọn "Just the push event"
5. Chọn "Active"

#### GitLab

1. Truy cập repository > Settings > Webhooks
2. URL: `http://your-jenkins-url/gitlab-webhook/`
3. Chọn "Push events" và chỉ định nhánh `main`
4. Chọn "Enable SSL verification"

### 5. Tạo Jenkins Pipeline

1. Tạo một Jenkins job mới, chọn loại "Pipeline"
2. Cấu hình Pipeline:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: URL của repository
   - Branch Specifier: `*/main`
   - Script Path: `Jenkinsfile`
3. Cấu hình Build Triggers:
   - Chọn "GitHub hook trigger for GITScm polling" (nếu sử dụng GitHub)
   - Hoặc "GitLab webhook" (nếu sử dụng GitLab)

## Quy trình CI/CD

1. Developer commit code vào nhánh `main`
2. Webhook kích hoạt Jenkins pipeline
3. Jenkins checkout code từ repository
4. Jenkins kiểm tra cấu trúc dự án
5. Jenkins deploy code lên Firebase Hosting
6. Jenkins gửi thông báo kết quả qua Slack/Email

## Xử lý sự cố

### Lỗi deploy Firebase

Kiểm tra:
- File credentials Firebase có đúng không
- Project ID Firebase có đúng không
- Quyền truy cập Firebase có đủ không

### Lỗi webhook

Kiểm tra:
- URL webhook có đúng không
- Jenkins có thể truy cập từ internet không
- Cấu hình webhook có đúng không

## Tài liệu tham khảo

- [Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/)
- [Firebase Hosting](https://firebase.google.com/docs/hosting)
- [Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/)
