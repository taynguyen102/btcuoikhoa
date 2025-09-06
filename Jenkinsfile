pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t my-web-app .'
            }
        }
        stage('Run Docker Container') {
            steps {
                echo "Running Docker container..."
                sh 'docker run -d -p 9080:80 --name my-web-app my-web-app || true'
            }
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
