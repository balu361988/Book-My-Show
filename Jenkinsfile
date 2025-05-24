pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')  // Jenkins credential ID
        DOCKER_IMAGE = "balu361988/bms:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/balu361988/Book-My-Show.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                    rm -rf node_modules package-lock.json
                    npm install
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('bookmyshow-app') {
                    script {
                        sh "docker build -t ${DOCKER_IMAGE} ."
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                    echo "Updating deployment.yaml with image: ${DOCKER_IMAGE}"
                    sed -i "s|image: .*|image: ${DOCKER_IMAGE}|" deployment.yaml
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext to: 'kastrokiran@gmail.com',
                     subject: 'Jenkins Job SUCCESS: Book-My-Show',
                     body: 'Pipeline completed successfully ✅'
        }

        failure {
            emailext to: 'kastrokiran@gmail.com',
                     subject: 'Jenkins Job FAILURE: Book-My-Show',
                     body: 'Pipeline failed ❌'
        }
    }
}

