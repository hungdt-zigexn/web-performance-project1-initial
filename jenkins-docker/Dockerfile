FROM jenkins/jenkins:lts

USER root

# Cài đặt các công cụ cần thiết
RUN apt-get update && \
    apt-get install -y curl gnupg2 lsb-release python3-pip

# Cài đặt Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash - && \
    apt-get install -y nodejs

# Cài đặt Firebase CLI
RUN npm install -g firebase-tools@11.30.0

# Cài đặt các plugin Jenkins
RUN jenkins-plugin-cli --plugins \
    git \
    workflow-aggregator \
    pipeline-stage-view \
    blueocean \
    credentials-binding \
    email-ext \
    slack

# Trở lại user jenkins
USER jenkins
