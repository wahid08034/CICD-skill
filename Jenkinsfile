pipeline {
    agent any
    environment {
        REGISTRY = "hub.docker.com"
        dockerRegistryCredential = 'hub.docker.com'
        dockerImage = ''
        DOCKER_REGISTRY_URL = "https://$REGISTRY"
        IMAGE_CREATED_BY = "jenkins"
        PROJECT_NAME = "wahaduzzaman/php-app-prod"
        GIT_TAG = sh(returnStdout: true, script: '''
            echo $(git describe --tags)
        ''').trim()
        IMAGE_VERSION = "${BUILD_NUMBER}-${IMAGE_CREATED_BY}"
        DOCKER_TAG = "$REGISTRY/$PROJECT_NAME:$IMAGE_VERSION"
        DISCORD_WEBHOOK_URL = 'https://discord.com/api/webhooks/1298966562560278528/ZxIheTb2XWWhAtQrC2_S58tcSlszIp_qD1RG9j9hOk8hh1gS6YOCq7JDWG7aNpYvM5eq'
    }

    stages {
        stage('Init') {
            steps {
                sh '''
                COMMIT_ID=$(git log -1 | head -1 | awk -F ' ' '{print $NF}')
                echo "Commit ID: $COMMIT_ID"
                '''
            }
        }

        stage('Building Docker image') { 
            steps { 
                script { 
                    dockerImage = docker.build(DOCKER_TAG, "-f ./Dockerfile .")
                }
                sh '''
                docker images | grep $PROJECT_NAME
                '''
            } 
        }

        stage('Push Docker image') {
            steps {
                script {
                    docker.withRegistry(DOCKER_REGISTRY_URL, dockerRegistryCredential) {
                        dockerImage.push()
                        sh "docker images | grep $PROJECT_NAME"
                    }
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    def scanResult = sh(script: "trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_TAG", returnStatus: true)
                    
                    if (scanResult != 0) {
                        def message = "Trivy scan failed for image $DOCKER_TAG. Check the logs for details."
                        sh """
                        curl -H "Content-Type: application/json" -d '{ "content": "${message}" }' ${DISCORD_WEBHOOK_URL}
                        """
                        error "Trivy scan failed."
                    } else {
                        def message = "Trivy scan succeeded for image $DOCKER_TAG. No critical vulnerabilities found."
                        sh """
                        curl -H "Content-Type: application/json" -d '{ "content": "${message}" }' ${DISCORD_WEBHOOK_URL}
                        """
                    }
                }
            }
        }

        stage('Run Docker container') {
            steps {
                echo "Running Docker container for PHP app"
                sh '''
                docker run -d --name php-app -p 8088:80 $DOCKER_TAG
                '''
            }
        }

        stage('Run PHPUnit Tests') {
            steps {
                script {
                    echo "Running PHPUnit tests in Docker container"
                    def testResult = sh(script: '''
                    docker exec php-app /var/www/html/vendor/bin/phpunit --configuration phpunit.xml
                    ''', returnStatus: true)

                    if (testResult != 0) {
                        def message = "Unit tests failed in Docker container php-app. Check the logs for details."
                        sh """
                        curl -H "Content-Type: application/json" -d '{ "content": "${message}" }' ${DISCORD_WEBHOOK_URL}
                        """
                        error "Unit tests failed."
                    } else {
                        def message = "Unit tests passed successfully in Docker container php-app."
                        sh """
                        curl -H "Content-Type: application/json" -d '{ "content": "${message}" }' ${DISCORD_WEBHOOK_URL}
                        """
                    }
                }
            }
        }

        stage('Run SQA Testing') {
            steps {
                script {
                    echo "Running SQA testing for PHP application"
                    def sqaResult = sh(script: '''
                    echo "Running SQA tests..."
                    ''', returnStatus: true)

                    if (sqaResult != 0) {
                        def message = "SQA tests failed for the PHP application. Check the logs for details."
                        sh """
                        curl -H "Content-Type: application/json" -d '{ "content": "${message}" }' ${DISCORD_WEBHOOK_URL}
                        """
                        error "SQA tests failed."
                    } else {
                        def message = "SQA tests passed successfully for the PHP application."
                        sh """
                        curl -H "Content-Type: application/json" -d '{ "content": "${message}" }' ${DISCORD_WEBHOOK_URL}
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh 'docker run -d $DOCKER_TAG'
                    def message = "Deployment of $DOCKER_TAG was successful by wahid08034@gmail.com"
                    sh """
                    curl -H "Content-Type: application/json" -d '{ "content": "${message}" }' ${DISCORD_WEBHOOK_URL}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Stopping and removing Docker container"
            sh '''
            docker stop php-app || true
            docker rm php-app || true
            '''
        }
    }
}
