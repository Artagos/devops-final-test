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
                VM1_USER = 'laborant'
                VM1_PORT = '4444'
            }
            steps {
                sshagent(['ssh-key']) {
                    sh '''
                        echo "Deploying to VM-1 (${VM1_HOST}) ..."

                        TARGET_DIR="/opt/devops-final-test"

                         ssh -o StrictHostKeyChecking=no ${VM1_USER}@${VM1_HOST} "sudo mkdir -p ${TARGET_DIR}"
                         scp package.json index.js systemd/devops-final-test.service ${VM1_USER}@${VM1_HOST}:/tmp/
                         ssh -o StrictHostKeyChecking=no ${VM1_USER}@${VM1_HOST} "
                           sudo mv /tmp/package.json /tmp/index.js ${TARGET_DIR}/ &&
                           sudo mv /tmp/devops-final-test.service /etc/systemd/system/ &&
                           cd ${TARGET_DIR} &&
                           npm install --production &&
                           sudo systemctl enable devops-final-test &&
                           sudo systemctl daemon-reload &&
                           sudo systemctl restart devops-final-test
                         "

                        echo "Deployment to VM-1 completed (dry-run)"
                    '''
                }
            }
        }

        stage('Deploy docker VM') {
            environment {
                VM2_HOST = 'docker'
                VM2_USER = 'laborant'
                VM2_PORT = '4444'
            }
            steps {
                sshagent(['ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no laborant@docker \
                            "docker pull ttl.sh/artagos:2h && \
                             docker stop go-server || true && \
                             docker rm go-server || true && \
                             docker run -d -p 4444:4444 --name go-server ttl.sh/artagos:2h"
                    '''
                }
            }
        }
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
