pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Checking out source code..."
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                echo "⚙️ Building application with Maven..."
                sh '''
                mvn clean package -DskipTests
                ls -l target/
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo "🐳 Building Docker image..."
                sh 'docker build -t simple-web:latest .'
            }
        }

        stage('Deploy Application') {
            steps {
                echo "🚀 Running application container..."
                sh '''
                docker stop simple-web || true
                docker rm simple-web || true
                docker run -d --name simple-web -p 8080:8080 simple-web:latest
                '''
            }
        }

        stage('Deploy Nginx Proxy') {
            steps {
                echo "🌐 Deploying Nginx reverse proxy..."
                sh '''
                docker stop nginx || true
                docker rm nginx || true
                docker run -d --name nginx \
                  --link simple-web:simple-web \
                  -p 80:80 my-nginx
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs for details."
        }
    }
}

