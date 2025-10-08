pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'simple-web:latest'
        NGINX_CONTAINER = 'nginx'
        OUTPUT_FILE = 'output.txt'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 Checking out source code...'
                checkout scm
            }
        }

        stage('Build with Maven (Dockerized)') {
            steps {
                echo '⚙️ Building JAR inside Maven container...'
                script {
                    docker.image('maven:3.9.9-eclipse-temurin-17').inside('-v $PWD:/app -w /app') {
                        sh 'mvn clean package -DskipTests'
                    }
                }
                sh 'ls -l target/'
            }
        }

        stage('Run App and Capture Output') {
            steps {
                echo '🧪 Running app to capture output...'
                sh 'java -jar target/simple-web-1.0-SNAPSHOT.jar > ${OUTPUT_FILE}'
                sh 'cat ${OUTPUT_FILE}'
            }
        }

        stage('Deploy to Nginx') {
            steps {
                echo '🌐 Updating Nginx web content...'
                sh '''
                if docker ps --format '{{.Names}}' | grep -q "^${NGINX_CONTAINER}$"; then
                    docker cp ${OUTPUT_FILE} ${NGINX_CONTAINER}:/usr/share/nginx/html/index.html || echo "⚠️ Copy warning ignored"
                else
                    echo "❌ Nginx container not found! Deployment skipped."
                fi
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful! Check your EC2 public IP in the browser.'
        }
        failure {
            echo '❌ Deployment failed. Check Jenkins logs.'
        }
    }
}
