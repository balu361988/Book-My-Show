pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node 23'  // ✅ match the exact configured name in Jenkins global tools
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        FULL_IMAGE_NAME = 'balu361988/BMS:latest'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/balu361988/Book-My-Show.git'
                sh 'ls -la'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                      -Dsonar.projectName=BMS \
                      -Dsonar.projectKey=BMS \
                      -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Code Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Install NPM Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --out .', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivy.txt'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t BMS .'
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html BMS'
            }
        }

        stage('Tag & Push to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub') {
                        sh 'docker tag BMS ${FULL_IMAGE_NAME}'
                        sh 'docker push ${FULL_IMAGE_NAME}'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    echo "Using image: ${FULL_IMAGE_NAME}"

                    # Replace image in deployment file
                    sed -i 's|image: .*|image: ${FULL_IMAGE_NAME}|' deployment.yml

                    # Apply the Kubernetes deployment
                    kubectl apply -f deployment.yml --validate=false

                    # Wait for deployment to roll out
                    kubectl rollout status deployment/bms
                    """
                }
            }
        }
    }

    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}' Build Notification",
                body: """
                <b>Project:</b> ${env.JOB_NAME}<br/>
                <b>Build Number:</b> ${env.BUILD_NUMBER}<br/>
                <b>Status:</b> ${currentBuild.result}<br/>
                <b>Build URL:</b> <a href='${env.BUILD_URL}'>Click to View</a>
                """,
                to: 'kastrokiran@gmail.com',
                attachmentsPattern: 'trivy.txt,trivy-image-report.html'
        }
    }
}

