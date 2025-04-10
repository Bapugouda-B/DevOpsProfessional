pipeline {
    agent any

    environment {
        IMAGE_NAME = 'bapu12/my-app'
        K8S_MASTER_IP = '3.226.240.223'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Bapugouda-B/DevOpsProfessional.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME .'
                }
            }
        }

        stage('Run and Test App on Port 85') {
            steps {
                script {
                    sh '''
            echo "Checking for container using port 85..."
            CONTAINER_ID=$(docker ps --filter "publish=85" -q)

            if [ ! -z "$CONTAINER_ID" ]; then
              echo "Port 85 is already in use by container: $CONTAINER_ID"
              echo "Stopping and removing container..."
              docker rm -f $CONTAINER_ID || true
              sleep 3
            fi

            echo "Running test container on port 85..."
            docker run -d --name test-app -p 85:80 $IMAGE_NAME
            sleep 5

            echo "Running test case..."
            RESPONSE=$(curl -s http://localhost:85)
            echo "Response: $RESPONSE"

            if echo "$RESPONSE" | grep -q "<html"; then
                echo "Test passed."
                docker stop test-app && docker rm test-app
            else
                echo "Test failed. Stopping test-app container."
                docker stop test-app && docker rm test-app
                exit 1
            fi
            '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    echo "Deploying to Kubernetes from Jenkins..."

                    ssh -o StrictHostKeyChecking=no -i /home/ubuntu/.ssh/My_Key.pem ubuntu@$K8S_MASTER_IP << EOF
                      kubectl apply -f /home/ubuntu/DevOpsProfessional/deployment.yaml
                      kubectl apply -f /home/ubuntu/DevOpsProfessional/service.yaml
                    EOF
                    '''
                }
            }
        }
    }
}
