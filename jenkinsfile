pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/deepakramalingahia-boop/terraform.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t devops-webapp:latest .'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh '''
                    docker rm -f webapp || true
                    docker run -d -p 8081:80 --name webapp devops-webapp:latest
                    '''
                }
            }
        }
    }
}
