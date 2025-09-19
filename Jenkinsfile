pipeline {
    agent any

    tools {
        // Name must match the one configured in Global Tool Configuration
        maven 'Maven'
        sonarScanner 'SonarScanner'
    }

    environment {
        // Replace with your Docker Hub credentials ID
        DOCKER_HUB_CREDENTIALS = 'your-dockerhub-credentials-id' 
        // Replace with your Docker Hub username and image name
        DOCKER_IMAGE = 'your-dockerhub-username/your-app-name' 
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from Git...'
                // This step automatically pulls the code from the Git repository
                // configured in the job settings.
                checkout scm
            }
        }
        
        stage('Maven Build') {
            steps {
                echo 'Starting Maven build...'
                // The `-DskipTests` flag skips unit tests for a faster build
                // but you can remove it if you want to run tests.
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Running SonarQube analysis...'
                    // Replace 'My SonarQube Server' with the name you configured in Jenkins
                    withSonarQubeEnv('My SonarQube Server') {
                        // This command runs the SonarScanner
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        
        stage('Quality Gate Check') {
            steps {
                script {
                    echo 'Checking SonarQube Quality Gate status...'
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                    }
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    echo 'Building and pushing Docker image...'
                    def buildNumber = env.BUILD_NUMBER

                    // Build the image using the Dockerfile in the project root
                    sh "docker build -t ${DOCKER_IMAGE}:${buildNumber} ."
                    sh "docker tag ${DOCKER_IMAGE}:${buildNumber} ${DOCKER_IMAGE}:latest"

                    // Securely log in to Docker Hub using stored credentials
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    }

                    // Push the tagged images to Docker Hub
                    sh "docker push ${DOCKER_IMAGE}:${buildNumber}"
                    sh "docker push ${DOCKER_IMAGE}:latest"

                    echo 'Docker image pushed successfully.'
                }
            }
        }
    }
}
