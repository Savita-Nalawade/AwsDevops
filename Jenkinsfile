pipeline {
    agent any

    environment {
        REGISTRY = "my-docker-registry.com"
        IMAGE_NAME = "checkout-service"
        SONARQUBE_ENV = "sonarqube-server"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Savita-Nalawade/AwsDevops.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
                echo 'Artifact Creation Completed'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin $REGISTRY
                        docker build -t $REGISTRY/$IMAGE_NAME:${env.BUILD_NUMBER} .
                        docker push $REGISTRY/$IMAGE_NAME:${env.BUILD_NUMBER}
                    """
                }
            }
        }