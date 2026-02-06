pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'deepak37'          // Docker Hub username
        DOCKERHUB_CREDS = 'dockerhub-creds'  // Docker Hub credentials ID in Jenkins
        AWS_CREDS = 'aws-creds'              // AWS credentials ID in Jenkins
        EC2_USER = 'ubuntu'                  // EC2 SSH username
        SSH_KEY = '/path/to/private/key.pem' // Path to private key on Jenkins node
        EC2_HOST = 'YOUR_EC2_PUBLIC_IP'      // EC2 instance public IP
        AWS_REGION = 'us-east-1'             // AWS region
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Use credentials if private repo
                git branch: 'main', 
                    url: 'https://github.com/deepakramalingahia-boop/terraform.git'
                    // credentialsId: 'github-creds'  // Uncomment if repo is private
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Use first 7 chars of commit hash as image tag
                    def commit = env.GIT_COMMIT ?: sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.DOCKER_IMAGE = "devops-webapp:${commit}"
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDS}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker tag ${DOCKER_IMAGE} $USER/${DOCKER_IMAGE}"
                    sh "docker push $USER/${DOCKER_IMAGE}"
                }
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${AWS_CREDS}", usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    dir('.') {  // Assuming TF files are in repo root
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
