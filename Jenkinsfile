pipeline {
agent any


tools {
    maven 'maven'
}

environment {
    DOCKER_IMAGE = 'siddhantk03/food-frezty'
    IMAGE_TAG = "${BUILD_NUMBER}"
}

stages {

    stage('Checkout') {
        steps {
            echo 'Checking out source code...'
            checkout scm
        }
    }

    stage('Build JAR') {
        steps {
            echo 'Building Food-Frezty Spring Boot application...'

            sh '''
                mvn clean package -DskipTests
                ls -lh target/*.jar
            '''
        }
    }

    stage('Build Docker Image') {
        steps {
            echo "Building Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}"

            sh '''
                docker build \
                -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
            '''
        }
    }

    stage('Login to Docker Hub') {
        steps {
            echo 'Logging in to Docker Hub...'

            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                    echo "$DOCKER_PASS" | docker login \
                    -u "$DOCKER_USER" \
                    --password-stdin
                '''
            }
        }
    }

    stage('Push Docker Image') {
        steps {
            echo "Pushing ${DOCKER_IMAGE}:${IMAGE_TAG} to Docker Hub..."

            sh '''
                docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
            '''
        }
    }

    stage('Clean Docker System') {
        steps {
            echo 'Cleaning unused Docker resources...'

            sh '''
                docker image prune -f || true
            '''
        }
    }
}

post {
    success {
        echo "Food-Frezty image pushed successfully: ${DOCKER_IMAGE}:${IMAGE_TAG}"
    }

    failure {
        echo 'Food-Frezty CI pipeline failed.'
    }

    always {
        echo 'CI pipeline completed.'
    }
}
}
