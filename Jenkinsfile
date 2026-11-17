pipeline {
    // Runs on your remote EC2 instance where Maven and Docker are installed
    agent { label 'maven-agent' }

    /*tools {
        // Must match the exact names configured in Manage Jenkins > Tools
        maven 'Maven-3.9' 
        jdk 'JDK-17'
    }*/
        tools {
        maven 'Maven3'
        jdk 'JDK21'
    }


    environment {
        DOCKER_IMAGE = "your-dockerhub-username/your-app-name"
        SONAR_PROJECT_KEY = "your-sonarqube-project-key"
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
                // Matches the SonarQube Server name in Manage Jenkins > System
                withSonarQubeEnv('SonarQube-Local') {
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
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
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
