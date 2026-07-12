pipeline {

    agent any

    tools {
        maven 'Maven-3'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/Savita-Nalawade/AwsDevops.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=devsecops-app \
                    -Dsonar.projectName=devsecops-app
                    '''
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh '''
                trivy fs --format table -o trivy-fs-report.txt .
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t devsecops-app:latest .
                '''
            }
        }
        stage('ECR Login') {
    steps {
        withCredentials([
            [$class: 'AmazonWebServicesCredentialsBinding',
             credentialsId: 'aws-creds']
        ]) {
            sh '''
            aws ecr get-login-password --region us-east-1 | \
            docker login --username AWS --password-stdin \
            392900064210.dkr.ecr.us-east-1.amazonaws.com
            '''
        }
    }
}
stage('Push Image to ECR') {
    steps {
        sh '''
        docker tag devsecops-app:latest \
        392900064340.dkr.ecr.us-east-1.amazonaws.com/devsecops-app:latest

        docker push \
        392900064340.dkr.ecr.us-east-1.amazonaws.com/devsecops-app:latest
        '''

    }

}

}


    post {
        always {
            archiveArtifacts artifacts: 'trivy-fs-report.txt'
        }
    }
}