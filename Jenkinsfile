pipeline {
  agent any

  environment {
    IMAGE_NAME = "simple-web:latest"
    CONTAINER_NAME = "simple-web-app"
    NETWORK = "ci-cd-network"
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
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps {
        echo "🏗️ Building project with Maven..."
        sh 'mvn -B clean package'
      }
    }

    stage('Docker Build') {
      steps {
        echo "🐳 Building Docker image..."
        sh "docker build -t ${IMAGE_NAME} ."
      }
    }

    stage('Deploy Application') {
      steps {
        echo "🚀 Deploying application container..."
        sh '''
          # Create network if not exists
          docker network ls | grep ${NETWORK} || docker network create ${NETWORK}

          # Stop and remove old container if running
          docker rm -f ${CONTAINER_NAME} || true

          # Run the new container
          docker run -d --name ${CONTAINER_NAME} --network ${NETWORK} -p 8081:8080 ${IMAGE_NAME}
        '''
      }
    }

    stage('Deploy Nginx Proxy') {
      steps {
        echo "🌐 Deploying Nginx reverse proxy..."
        sh '''
          mkdir -p $WORKSPACE/nginx
          cat > $WORKSPACE/nginx/default.conf <<'EOF'
          server {
              listen 80;
              server_name _;

              location / {
                  proxy_pass http://${CONTAINER_NAME}:8080/;
                  proxy_set_header Host $host;
                  proxy_set_header X-Real-IP $remote_addr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  proxy_set_header X-Forwarded-Proto $scheme;
              }
          }
          EOF

          docker rm -f nginx-proxy || true
          docker run -d --name nginx-proxy --network ${NETWORK} -p 80:80 \
            -v $WORKSPACE/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro nginx:latest
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Deployment successful! Visit http://<your-public-ip>/"
    }
    failure {
      echo "❌ Deployment failed. Check Jenkins logs for details."
    }
  }
}

