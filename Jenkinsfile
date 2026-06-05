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
                stage('Deploy to target VM') {
            environment {
                VM1_HOST = 'target'
                VM1_USER = 'lab'
                VM1_PORT = '4444'
            }
            steps {
                sshagent(['ssh-key']) {
                    sh '''
                        echo "Deploying to VM-1 (${VM1_HOST}) ..."

                        TARGET_DIR="/opt/devops-final-test"

                         ssh -o StrictHostKeyChecking=no ${VM1_USER}@${VM1_HOST} "mkdir -p ${TARGET_DIR}"
                         scp package.json index.js ${VM1_USER}@${VM1_HOST}:${TARGET_DIR}/
                         ssh -o StrictHostKeyChecking=no ${VM1_USER}@${VM1_HOST} "
                           cd ${TARGET_DIR} &&
                           npm install --production &&
                           sudo systemctl daemon-reload &&
                           sudo systemctl restart devops-final-test
                         "

                        echo "Deployment to VM-1 completed (dry-run)"
                    '''
                }
            }
        }

        // stage('Deploy docker VM') {
        //     environment {
        //         VM2_HOST = '192.168.1.102'
        //         VM2_USER = 'deploy'
        //         VM2_PORT = '4444'
        //     }
        //     steps {
        //         sshagent(['vm2-ssh-key']) {
        //             sh '''
        //                 echo "Deploying to VM-2 (${VM2_HOST}) ..."

        //                 TARGET_DIR="/opt/devops-final-test"

        //                 # TODO: uncomment and adjust when VM-2 is reachable
        //                 # ssh ${VM2_USER}@${VM2_HOST} "mkdir -p ${TARGET_DIR}"
        //                 # scp package.json index.js ${VM2_USER}@${VM2_HOST}:${TARGET_DIR}/
        //                 # ssh ${VM2_USER}@${VM2_HOST} "
        //                 #   cd ${TARGET_DIR} &&
        //                 #   npm install --production &&
        //                 #   sudo systemctl daemon-reload &&
        //                 #   sudo systemctl restart devops-final-test
        //                 # "

        //                 echo "Deployment to VM-2 completed (dry-run)"
        //             '''
        //         }
        //     }
        // }
        // stage('Deploy to Kubernetes') {
        //     steps {
        //         sh '''
        //             echo "Deploying ${IMAGE_URI} to namespace ${K8S_NAMESPACE} ..."

        //             # TODO: replace with actual kubeconfig context
        //             # kubectl config use-context <your-cluster-context>

        //             # kubectl -n ${K8S_NAMESPACE} set image deployment/${IMAGE_NAME} \\
        //             #   ${IMAGE_NAME}=${IMAGE_URI}

        //             echo "Deployment to Kubernetes completed (dry-run)"
        //         '''
        //     }
        // }
    }

    post {
        always {
            echo "Pipeline finished. Status: ${currentBuild.result}"
        }
    }
}
