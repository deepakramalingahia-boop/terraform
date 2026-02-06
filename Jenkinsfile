pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'deepak37'          // Docker Hub username
        EC2_USER = 'ubuntu'                                 // EC2 SSH username
        SSH_KEY = '/path/to/private/key.pem'               // Path to EC2 private key on Jenkins node
        AWS_REGION = 'us-east-1'                           // Your AWS region
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/deepakramalingahia-boop/terraform.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Use first 7 chars of commit hash as image tag
                    env.DOCKER_IMAGE = "devops-webapp:${env.GIT_COMMIT.take(7)}"
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker tag ${DOCKER_IMAGE} $USER/${DOCKER_IMAGE}"
                    sh "docker push $USER/${DOCKER_IMAGE}"
                }
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                // Inject AWS credentials into Terraform
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    dir('terraform') {  // Make sure your TF files are inside 'terraform/' folder
                        sh 'terraform init'
                        sh 'terraform apply -auto-approve'
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${EC2_USER}@${EC2_HOST} '
                    docker rm -f webapp || true
                    docker pull ${DOCKERHUB_USER}/${DOCKER_IMAGE}
                    docker run -d -p 80:80 --name webapp ${DOCKERHUB_USER}/${DOCKER_IMAGE}
                '
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline finished!"
        }
        failure {
            echo "Pipeline failed! Check logs."
        }
    }
}

