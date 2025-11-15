pipeline {
    agent any
    
    environment {
        REGISTRY = 'flamorz/gs2-gateway'
        CONTAINER = 'gs2-gateway'
        DOCKER_TAG = "1.$BUILD_NUMBER"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Docker Cleanup') {
            steps {
                script {
                    // Cleanup existing container and image
                    sh """
                    if command -v docker &> /dev/null; then
                        docker ps -q -f name=$CONTAINER | xargs -r docker stop
                        docker ps -a -q -f name=$CONTAINER | xargs -r docker rm
                        docker images -q $REGISTRY | xargs -r docker rmi -f
                    fi
                    """
                }
            }
        }

        stage('Docker Build Image') {
            steps {
                script {
                    sh """
                        docker build -t "$REGISTRY:$DOCKER_TAG" \
                            -f ModelChanger.Gateway/Dockerfile \
                            ModelChanger
                    """
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker_cred',
                            usernameVariable: 'USERNAME',
                            passwordVariable: 'PASSWORD'
                        )
                    ]) {
                        sh """
                        docker login -u $USERNAME -p $PASSWORD
                        docker push $REGISTRY:$DOCKER_TAG
                        """
                    }
                }
            }
        }

        stage('Deploy New Container') {
            steps {
                script {
                    sh """
                    docker run --restart=always \
                        --name $CONTAINER \
                        --network fiap \
                        -d -p 7000:8080/tcp $REGISTRY:$DOCKER_TAG
                    """
                }
            }
        }
    }
}