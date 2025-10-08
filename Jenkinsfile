pipeline {
    agent any

    environment {
        APP_IMAGE = 'simple-web:latest'
        NGINX_IMAGE = 'my-nginx:latest'
        WORKDIR = "${env.WORKSPACE}"  // <-- absolute path inside Jenkins container
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
                    args "-v ${env.WORKSPACE}:${env.WORKSPACE} -w ${env.WORKSPACE}"
                }
            }
            steps {
                echo "⚙️ Building application with Maven inside Docker..."
                sh '''
                mvn clean package -DskipTests
                echo "Contents of target folder:"
                ls -l target/
                '''
            }
        }

        stage('Docker Build - App') {
            steps {
                echo "🐳 Building Docker image for simple-web..."
                sh '''
                echo "Building image using JAR at: ${WORKDIR}/target/simple-web-1.0-SNAPSHOT.jar"
                docker build -t ${APP_IMAGE} -f Dockerfile ${WORKDIR}
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
                echo "🌐 Building and running Nginx reverse proxy..."
                sh '''
                # Create nginx.conf if not exists
                if [ ! -f nginx.conf ]; then
                    cat <<'EOF' > nginx.conf
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
                fi

                # Create Dockerfile.nginx if not exists
                if [ ! -f Dockerfile.nginx ]; then
                    cat <<'EOF' > Dockerfile.nginx
                    FROM nginx:latest
                    COPY nginx.conf /etc/nginx/nginx.conf
EOF
                fi

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
            echo "✅ Deployment successful! Visit your EC2 public IP in the browser."
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs for details."
        }
    }
}

