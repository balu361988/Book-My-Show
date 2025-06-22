pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        FULL_IMAGE_NAME = 'balu361988/bms:latest'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/KastroVKiran/Book-My-Show.git'
                sh 'ls -la'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=BMS \
                        -Dsonar.projectKey=BMS
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    sleep(time: 10, unit: 'SECONDS') // Avoid race condition
                    timeout(time: 10, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    cd bookmyshow-app
                    ls -la
                    if [ -f package.json ]; then
                        rm -rf node_modules package-lock.json
                        npm install
                    else
                        echo "Error: package.json not found in bookmyshow-app!"
                        exit 1
                    fi
                '''
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                sh '''
                    echo "Running OWASP Dependency Check..."
                    mkdir -p dependency-check-report
                    dependency-check.sh --project "BookMyShow" \
                                        --scan bookmyshow-app \
                                        --out dependency-check-report \
                                        --format HTML || true
                '''
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt || true'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub') {
                        sh '''
                            echo "Building Docker image..."
                            docker build --no-cache -t ${FULL_IMAGE_NAME} -f bookmyshow-app/Dockerfile bookmyshow-app

                            echo "Pushing Docker image to Docker Hub..."
                            docker push ${FULL_IMAGE_NAME}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                    echo "Stopping and removing old container if it exists..."
                    docker stop bms || true
                    docker rm bms || true

                    echo "Running new container..."
                    docker run -d --restart=always --name bms -p 3000:3000 ${FULL_IMAGE_NAME}

                    echo "Checking running containers..."
                    docker ps -a

                    echo "Container Logs:"
                    sleep 5
                    docker logs bms
                '''
            }
        }
    }

    post {
        always {
            emailext (
                attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                    Project: ${env.JOB_NAME}<br/>
                    Build Number: ${env.BUILD_NUMBER}<br/>
                    URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a><br/>
                """,
                to: 'kastrokiran@gmail.com',
                attachmentsPattern: 'trivyfs.txt, dependency-check-report/dependency-check-report.html'
            )
        }
    }
}

