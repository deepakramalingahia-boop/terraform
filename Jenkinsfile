pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "deepak37"
        EC2_USER = "ubuntu"
        SSH_KEY = "/path/to/private/key.pem"  // Jenkins node must have access
        AWS_REGION = "us-east-1"             // Your AWS region
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/deepakramalingahia-boop/terraform.git'
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    withEnv(["AWS_REGION=${AWS_REGION}"]) {
                        sh 'terraform init'
                        sh 'terraform apply -auto-approve'
                    }
                }
            }
        }

        stage('Get EC2 Public IP') {
            steps {
                dir('terraform') {
                    script {
                        EC2_HOST = sh(
                            returnStdout: true,
                            script: "terraform output -raw ec2_public_ip"
                        ).trim()
                        echo "EC2 Public IP: ${EC2_HOST}"
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                echo "Running unit tests..."
                # Add your actual test commands here
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    DOCKER_IMAGE = "devops-webapp:${env.GIT_COMMIT.take(7)}"
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker tag ${DOCKER_IMAGE} $USER/${DOCKER_IMAGE}
                    docker push $USER/${DOCKER_IMAGE}
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${EC2_USER}@${EC2_HOST} '
                    docker stop webapp || true
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
            echo "Pipeline failed! Check the logs."
        }
    }
}

