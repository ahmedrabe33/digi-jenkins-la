pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        GITHUB_REPO = 'https://github.com/antoniosmounir33/digi-jenkins-lab.git'
        DOCKERHUB_USERNAME = 'antoniosmounir222000'
        IMAGE_NAME = 'service-app'
        IMAGE_TAG = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: "${GITHUB_REPO}",
                        credentialsId: 'github-pat-creds'
                    ]]
                ])
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        "$SCANNER_HOME/bin/sonar-scanner" \
                          -Dsonar.projectKey=service-app \
                          -Dsonar.projectName=service-app \
                          -Dsonar.sources=src \
                          -Dsonar.host.url="$SONAR_HOST_URL" \
                          -Dsonar.login="$SONAR_AUTH_TOKEN" \
                          -Dsonar.qualitygate.wait=true
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t "$IMAGE_TAG" .
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    trivy image \
                      --exit-code 1 \
                      --severity CRITICAL \
                      --ignore-unfixed \
                      --no-progress \
                      "$IMAGE_TAG"
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push "$IMAGE_TAG"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f service-app || true

                    docker run -d \
                      --name service-app \
                      -p 8081:8080 \
                      "$IMAGE_TAG" \
                      php -S 0.0.0.0:8080 -t .
                '''
            }
        }
    }

    post {
        always {
            sh '''
                docker rmi "$IMAGE_TAG" || true
                docker image prune -f || true
            '''
        }

        success {
            echo 'Lab 2 pipeline completed successfully.'
        }

        failure {
            echo 'Lab 2 pipeline failed. Check Console Output.'
        }
    }
}
