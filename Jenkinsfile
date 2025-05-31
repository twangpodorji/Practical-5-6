pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS 24.0.2'
    }
    
    environment {
        CI = 'true'
        NODE_ENV = 'production'
        BUILD_DOCKER = 'true'
        DOCKER_IMAGE = 'wangpo1642/dso101-5-6'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh '''
                    echo "Running React tests..."
                    npm test
                    echo "Tests completed successfully"
                '''
            }
            post {
                always {
                    echo 'Test stage completed'
                }
                success {
                    echo 'All tests passed!'
                }
                failure {
                    echo 'Some tests failed!'
                }
            }
        }
        
        stage('Build Application') {
            steps {
                echo 'Building React application...'
                sh 'npm run build'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'build/**/*', allowEmptyArchive: true
                }
            }
        }
        
        stage('Build Docker Image') {
            when {
                anyOf {
                    branch 'main'
                    environment name: 'BUILD_DOCKER', value: 'true'
                }
            }
            steps {
                echo 'Building Docker image...'
                script {
                    try {
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                        echo "Docker image built successfully"
                    } catch (Exception e) {
                        echo "Docker build failed: ${e.getMessage()}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            when {
                anyOf {
                    branch 'main'
                    environment name: 'BUILD_DOCKER', value: 'true'
                }
            }
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Clean Docker Images') {
            when {
                anyOf {
                    branch 'main'
                    environment name: 'BUILD_DOCKER', value: 'true'
                }
            }
            steps {
                echo 'Cleaning up local Docker images...'
                sh """
                    docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                    docker rmi ${DOCKER_IMAGE}:latest || true
                    docker system prune -f || true
                """
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                if (env.BRANCH_NAME == 'main') {
                    sh 'npm run deploy:prod'
                } else {
                    sh 'npm run deploy:stage'
                }
                }
            }
	    }

        stage('Run Tests') {
            steps {
                echo 'Skipping tests...'
            }
        }

    }
    
    post {
        always {
            echo 'Pipeline completed!'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            script {
                if (env.BRANCH_NAME == 'main') {
                    echo "Docker image pushed: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}