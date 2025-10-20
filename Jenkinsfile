// Jenkinsfile
pipeline {
    agent any

    tools {
        // Use the NodeJS tool you configured in Jenkins Global Tools
        nodejs 'NodeJS-7.8.0'
    }

    environment {
        // Define environment variables conditionally based on the branch name.
        // This is the core logic for handling differences between environments.
        APP_PORT = ''
        DOCKER_TAG = ''
        IMAGE_NAME = ''
    }

    stages {
        stage('Initialize Environment') {
            steps {
                script {
                    // Jenkins provides the BRANCH_NAME environment variable automatically
                    if (env.BRANCH_NAME == 'main') {
                        echo "Running on MAIN branch. Setting production configs."
                        APP_PORT = '3000'
                        DOCKER_TAG = 'v1.0'
                        IMAGE_NAME = 'nodemain'
                    } else if (env.BRANCH_NAME == 'dev') {
                        echo "Running on DEV branch. Setting development configs."
                        APP_PORT = '3001'
                        DOCKER_TAG = 'v1.0'
                        IMAGE_NAME = 'nodedev'
                    } else {
                        // Handle other branches if necessary
                        echo "Branch is not main or dev. Skipping deployment-specific setup."
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                // Clones the source code from the repository
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Install NodeJS dependencies
                echo 'Building the application...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                // Run unit tests
                echo 'Testing the application...'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image using the dynamic name and tag
                echo "Building Docker image ${IMAGE_NAME}:${DOCKER_TAG}"
                sh "docker build -t ${IMAGE_NAME}:${DOCKER_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Logic to ensure minimal downtime by stopping the old container
                    // only after the new image is ready.
                    echo "Deploying application to port ${APP_PORT}"
                    
                    // Check if a container with the same name is running
                    def runningContainer = sh(script: "docker ps -q --filter name=${IMAGE_NAME}", returnStdout: true).trim()
                    
                    if (runningContainer) {
                        echo "Stopping and removing existing container ${runningContainer}"
                        sh "docker stop ${runningContainer}"
                        sh "docker rm ${runningContainer}"
                    } else {
                        echo "No existing container found. Proceeding with new deployment."
                    }
                    
                    // Run the new container
                    echo "Starting new container..."
                    sh "docker run -d --name ${IMAGE_NAME} -p ${APP_PORT}:3000 ${IMAGE_NAME}:${DOCKER_TAG}"
                }
            }
        }
    }
}
