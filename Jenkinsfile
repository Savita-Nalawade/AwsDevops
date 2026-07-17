pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "savitanalawade"
        IMAGE_NAME = "docker-image"
        SONARQUBE_ENV = "sonarqube-server"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'dev',
                    url: 'https://github.com/Savita-Nalawade/AwsDevops.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
                echo 'Build Completed'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
                echo 'Unit Tests Completed'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest .
                """
                echo 'Docker Image Build Completed'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                trivy image ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push To Docker Hub') {
            steps {
                withCredentials([
                    string(credentialsId: 'dockerhubCred',
                    variable: 'dockerhubCred')
                ]) {

                    sh """
                    docker login -u ${DOCKERHUB_USERNAME} -p ${dockerhubCred}
                    docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest
                    """
                }
            }


    post {
        success {
            echo 'Pipeline Executed Successfully'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}