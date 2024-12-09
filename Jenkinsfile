pipeline {
    agent any
    environment {
        REGISTRY = "hub.docker.com"
        dockerRegistryCredential = 'docker.course.nxt'
        dockerImage = ''
        DOCKER_REGISTRY_URL = "https://$REGISTRY"
        IMAGE_CREATED_BY = "jenkins"
        PROJECT_NAME = "php-app-prod"
    }
    stages {
        stage('Init') {
            steps {
                script {
                    // Attempt to retrieve the Git tag; fallback to "no-tag" if none exist
                    GIT_TAG = sh(
                        script: '''git describe --tags || echo "no-tag"''',
                        returnStdout: true
                    ).trim()

                    // Build the image version and Docker tag
                    IMAGE_VERSION = "${BUILD_NUMBER}-${IMAGE_CREATED_BY}-${GIT_TAG}"
                    DOCKER_TAG = "${REGISTRY}/${PROJECT_NAME}:${IMAGE_VERSION}"

                    // Print the resolved values
                    echo "Git Tag: ${GIT_TAG}"
                    echo "Docker Tag: ${DOCKER_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build(DOCKER_TAG, "-f ./Dockerfile .")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("$DOCKER_REGISTRY_URL", dockerRegistryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
