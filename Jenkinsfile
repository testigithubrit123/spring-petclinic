pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "vinayak18/spring-petclinic:latest"
        HOST_PORT = "8081"   // Change this if needed
        CONTAINER_PORT = "8080"
    }
    stages {

        stage('Clean Old Containers') {
            steps {
                echo "Stopping any old Spring PetClinic containers..."
                sh """
                    docker ps -q --filter "ancestor=${DOCKER_IMAGE}" | xargs -r docker stop
                    docker ps -q --filter "ancestor=${DOCKER_IMAGE}" | xargs -r docker rm
                """
            }
        }

        stage('Build with Maven') {
            steps {
                echo "Building Spring PetClinic project..."
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Run Docker Container') {
            steps {
                echo "Running Docker container..."
                sh """
                    docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} ${DOCKER_IMAGE}
                """
            }
        }
    }

    post {
        success {
            echo "Spring PetClinic is running at http://<VM-IP>:${HOST_PORT}"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
 
