pipeline {
    agent any 

    parameters {
        string(name: 'VERSION', description: 'Enter the APP Version')
    }

    environment {
        AWS_ACCOUNT_ID = "372123585601"
        REGION = "ap-northeast-3"
        REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/devops"
        DOCKER_IMAGE = "zepto-app-web:${VERSION}"
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REGISTRY_CREDENCIALS = "docker_creds"
    }

    stages {
        stage('Clone') {
            steps {
                echo "Cloing the GitHub Repository"
                git url: 'https://github.com/Priyakirana/cartify-me.git', branch: 'main'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building the Docker Image ${DOCKER_IMAGE}"
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws_creds', region: "${REGION}") {
                        echo "Pushing the docker image to AWS ECR"
                        sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REPO_URI}
                            docker tag ${DOCKER_IMAGE} ${REPO_URI}:${VERSION}
                            docker push ${REPO_URI}:${VERSION}
                        """
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDENCIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                  echo "Pushing the docker image to Docker Hub"
                  sh """
                    docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                    docker tag ${DOCKER_IMAGE} priyakirana14/${DOCKER_IMAGE}
                    docker push priyakirana14/${DOCKER_IMAGE}
                    """  
            }
            }
        }

        stage("RUN Docker Compose")
        {
            steps {
                echo "Start the Containers using docker-compose"
                sh "docker-compose up -d"
            }
        post {
            success {
                echo "Docker Containers are started sucessfully"
            }
            failure {
                echo "Docker Containers are Failed"
            }
        }
        }
    }
}
