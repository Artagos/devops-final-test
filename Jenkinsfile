pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'ttl.sh'
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
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying ${IMAGE_URI} to namespace ${K8S_NAMESPACE} ..."

                    # TODO: replace with actual kubeconfig context
                    # kubectl config use-context <your-cluster-context>

                    # kubectl -n ${K8S_NAMESPACE} set image deployment/${IMAGE_NAME} \\
                    #   ${IMAGE_NAME}=${IMAGE_URI}

                    echo "Deployment to Kubernetes completed (dry-run)"
                '''
            }
        }

        stage('Deploy to VM-1') {
            environment {
                VM1_HOST = '192.168.1.101'
                VM1_USER = 'deploy'
                VM1_PORT = '4444'
            }
            steps {
                sshagent(['vm1-ssh-key']) {
                    sh '''
                        echo "Deploying to VM-1 (${VM1_HOST}) ..."

                        # TODO: uncomment and adjust when VM-1 is reachable
                        # ssh ${VM1_USER}@${VM1_HOST} "
                        #   docker pull ${IMAGE_URI} &&
                        #   docker stop ${IMAGE_NAME} 2>/dev/null; docker rm ${IMAGE_NAME} 2>/dev/null;
                        #   docker run -d --name ${IMAGE_NAME} -p ${VM1_PORT}:4444 ${IMAGE_URI}
                        # "

                        echo "Deployment to VM-1 completed (dry-run)"
                    '''
                }
            }
        }

        stage('Deploy to VM-2') {
            environment {
                VM2_HOST = '192.168.1.102'
                VM2_USER = 'deploy'
                VM2_PORT = '4444'
            }
            steps {
                sshagent(['vm2-ssh-key']) {
                    sh '''
                        echo "Deploying to VM-2 (${VM2_HOST}) ..."

                        # TODO: uncomment and adjust when VM-2 is reachable
                        # ssh ${VM2_USER}@${VM2_HOST} "
                        #   docker pull ${IMAGE_URI} &&
                        #   docker stop ${IMAGE_NAME} 2>/dev/null; docker rm ${IMAGE_NAME} 2>/dev/null;
                        #   docker run -d --name ${IMAGE_NAME} -p ${VM2_PORT}:4444 ${IMAGE_URI}
                        # "

                        echo "Deployment to VM-2 completed (dry-run)"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Status: ${currentBuild.result}"
        }
    }
}
