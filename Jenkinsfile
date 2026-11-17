pipeline {
    // Runs on your remote EC2 instance where Maven and Docker are installed
    agent { label 'maven-agent' }

    tools {
        maven 'Maven3'
        jdk 'JDK21'
    }

    environment {
        // Replace 'your-dockerhub-username' with your actual Docker Hub username
        DOCKER_IMAGE      = "monish2703/simple-docker-webapp"
        SONAR_PROJECT_KEY = "simple-docker-webapp"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Pulls the code from your forked GitHub repository
                checkout scm
            }
        }

        stage('Build & Test with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                // Updated to match your exact SonarQube Server name in Jenkins
                withSonarQubeEnv('Local-SonarQube') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY}
                    """
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                // Pauses pipeline until SonarQube sends the webhook result back via ngrok
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker logout
                    """
                }
            }
        }
    }

    post {
        always {
            // Cleans up local Docker images on the EC2 agent to save disk space
            sh "docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} || true"
        }
    }
}
