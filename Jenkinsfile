pipeline {
    agent any

    environment {
        APP_IMAGE = 'simple-web:latest'
        NGINX_IMAGE = 'my-nginx:latest'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Checking out source code..."
                checkout scm
            }
        }

        stage('Build with Maven') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17'
                    args "-v \"${env.WORKSPACE}\":/workspace -w /workspace"
                }
            }
            steps {
                echo "⚙️ Building application with Maven..."
                sh '''
                mvn clean package -DskipTests
                echo "✅ Contents of target folder:"
                ls -l target/
                '''
            }
        }

        stage('Docker Build - App') {
            steps {
                echo "🐳 Building Docker image for simple-web..."
                sh '''
                echo "📦 Preparing Docker build context..."
                mkdir -p docker-build
                cp -r target docker-build/
                cp Dockerfile docker-build/

                echo "🚧 Building Docker image from docker-build/..."
                cd docker-build
                docker build -t ${APP_IMAGE} .
                '''
            }
        }

        stage('Deploy Application Container') {
            steps {
                echo "🚀 Running simple-web container..."
                sh '''
                docker stop simple-web || true
                docker rm simple-web || true
                docker run -d --name simple-web -p 8080:8080 ${APP_IMAGE}
                '''
            }
        }

        stage('Build & Deploy Nginx Proxy') {
            steps {
                echo "🌐 Setting up Nginx reverse proxy..."
                sh '''
                cat > nginx.conf <<'EOF'
                events { }

                http {
                    upstream app_server {
                        server simple-web:8080;
                    }

                    server {
                        listen 80;

                        location / {
                            proxy_pass http://app_server;
                        }
                    }
                }
EOF

                cat > Dockerfile.nginx <<'EOF'
                FROM nginx:latest
                COPY nginx.conf /etc/nginx/nginx.conf
EOF

                docker build -t ${NGINX_IMAGE} -f Dockerfile.nginx .
                docker stop nginx || true
                docker rm nginx || true
                docker run -d --name nginx \
                    --link simple-web:simple-web \
                    -p 80:80 ${NGINX_IMAGE}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Visit http://<your-ec2-public-ip>"
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs for details."
        }
    }
}

