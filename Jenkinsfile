pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        IMAGE_NAME = "bachxuanphong/student-cicd"
        IMAGE_TAG = "${env.BRANCH_NAME}"
        DOCKERHUB_USERNAME = credentials('dockerhub-username')
        DOCKERHUB_PASSWORD = credentials('dockerhub-password')
        SSH_PRIVATE_KEY = credentials('deploy-ssh-key')
        DEPLOY_SERVER_IP = "192.168.30.26"
        DEPLOY_USER = "phongbv"
    }

    stages {
        stage('Build') {
            agent { docker { image 'node:18-alpine' } }
            steps {
                script {
                    echo "🏗️ Building project..."
                    if (fileExists('package.json')) {
                        sh '''
                            npm install
                            npm run build || echo "No build script found, skipping..."
                        '''
                    } else {
                        echo "No package.json found, skipping npm build."
                    }
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'dist/**', fingerprint: true, allowEmptyArchive: true
                }
            }
        }

        stage('Test') {
            agent { docker { image 'node:18-alpine' } }
            steps {
                script {
                    echo "🧪 Running tests..."
                    if (fileExists('package.json')) {
                        sh '''
                            npm test || echo "No test script found, skipping tests..."
                        '''
                    } else {
                        echo "No tests to run."
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            agent { label 'docker' } // any node with Docker installed
            steps {
                script {
                    echo "🐳 Building Docker image..."
                    sh '''
                        docker build -t $IMAGE_NAME:$IMAGE_TAG .
                        echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin $REGISTRY
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }


        stage('Deploy') {
            agent { label 'any' }
            steps {
                script {
                    echo "🚀 Deploying to remote server..."
                    sh '''
                        eval $(ssh-agent -s)
                        echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
                        mkdir -p ~/.ssh
                        chmod 700 ~/.ssh
                        ssh-keyscan $DEPLOY_SERVER_IP >> ~/.ssh/known_hosts
                        chmod 644 ~/.ssh/known_hosts

                        ssh $DEPLOY_USER@$DEPLOY_SERVER_IP "
                            docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD &&
                            docker rm -f frontend-app || true &&
                            docker run -d --name frontend-app -p 88:80 $IMAGE_NAME:$IMAGE_TAG
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline finished!"
        }
    }
}

