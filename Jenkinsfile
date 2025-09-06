pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myapp:latest"
        REGISTRY = "myregistry.local"
        SERVERS = "221.132.18.36"  
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh """
                  echo "==> Building Docker image..."
                  docker build -t $DOCKER_IMAGE .
                """
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                sh """
                  echo "==> Tag & push image to registry"
                  docker tag $DOCKER_IMAGE $REGISTRY/$DOCKER_IMAGE
                  docker push $REGISTRY/$DOCKER_IMAGE
                """
            }
        }

        stage('Deploy to Staging/QA') {
            steps {
                sh """
                  echo "==> Deploy to staging servers..."
                  for server in $SERVERS; do
                    echo "Deploying to \$server (staging)..."
                    ssh user@\$server "docker pull $REGISTRY/$DOCKER_IMAGE && \
                                       docker stop myapp || true && \
                                       docker rm myapp || true && \
                                       docker run -d --name myapp -p 7080:80 $REGISTRY/$DOCKER_IMAGE"
                  done
                """
            }
        }

        stage('Approval Required') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        input message: "Do you want to deploy to Production?", ok: "Proceed"
                    }
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                sh """
                  echo "==> Deploy to production servers..."
                  for server in $SERVERS; do
                    echo "Deploying to \$server (production)..."
                    ssh user@\$server "docker pull $REGISTRY/$DOCKER_IMAGE && \
                                       docker stop myapp || true && \
                                       docker rm myapp || true && \
                                       docker run -d --name myapp -p 80:80 $REGISTRY/$DOCKER_IMAGE"
                  done
                """
            }
        }
    }
}
