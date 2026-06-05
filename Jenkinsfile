pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'devops-final-test'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        IMAGE_URI = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
        K8S_NAMESPACE = 'devops-final'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                    npm test
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_URI} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    echo "Pushing image ${IMAGE_URI} to ${DOCKER_REGISTRY} ..."
                    // TODO: uncomment when registry is available
                    // docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying ${IMAGE_URI} to namespace ${K8S_NAMESPACE} ..."

                    # TODO: replace with actual kubeconfig context
                    # kubectl config use-context <your-cluster-context>

                    # Apply deployment manifest
                    # kubectl -n ${K8S_NAMESPACE} set image deployment/${IMAGE_NAME} \\
                    #   ${IMAGE_NAME}=${IMAGE_URI}

                    echo "Deployment to Kubernetes completed (dry-run)"
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Status: ${currentBuild.result}"
        }
    }
}
