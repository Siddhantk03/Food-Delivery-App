pipeline {
    agent any

    parameters {
        choice(
            name: 'ACTION',
            choices: ['DEPLOY', 'REMOVE'],
            description: 'Choose DEPLOY to deploy or REMOVE to remove the application'
        )
    }

    tools {
        maven 'maven'
    }

    environment {
    APP_NAME = "food-frezty"
    DOCKER_IMAGE = "siddhantk03/food-frezty"
    IMAGE_TAG = "${BUILD_NUMBER}"
}
    

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('Build JAR') {
            when {
                expression {
                    params.ACTION == 'DEPLOY'
                }
            }

            steps {
                echo "Building Food-Frezty Application..."

                sh '''
                    mvn clean package -DskipTests
                    ls -lh target/*.jar
                '''
            }
        }

        stage('Build Docker Image') {
            when {
                expression {
                    params.ACTION == 'DEPLOY'
                }
            }

            steps {
                echo "Building Docker Image..."

                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            when {
                expression {
                    params.ACTION == 'DEPLOY'
                }
            }

            steps {
                echo "Pushing Image to Docker Hub..."

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

                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy Application') {
    when {
        expression {
            params.ACTION == 'DEPLOY'
        }
    }
    steps {
        echo "Deploying Food-Frezty Application..."

        sh '''
            docker compose down --remove-orphans || true
            docker compose up -d --build
        '''
    }
}

        stage('Remove Application') {
            when {
                expression {
                    params.ACTION == 'REMOVE'
                }
            }

            steps {
                echo "Removing Food-Frezty Application..."

                sh '''
                    docker compose down --remove-orphans
                '''
            }
        }
    }

    post {
        success {
            echo "Food-Frezty pipeline executed successfully."
        }

        failure {
            echo "Food-Frezty pipeline execution failed."
        }

        always {
            echo "Pipeline completed."
        }
    }
}