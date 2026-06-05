pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'ttl.sh'
        IMAGE_NAME = 'devops-final-test'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        IMAGE_URI = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
        K8S_NAMESPACE = 'devops-final'
        POD = 'myapp'
        KUBE_SERVER = 'https://kubernetes:6443'
        KUBE_CRED = 'kube-token'
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
                    docker push ${IMAGE_URI}
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
                ssh -o StrictHostKeyChecking=no ${VM2_USER}@${VM2_HOST} \
                    "docker pull ${IMAGE_URI} && \
                     docker stop ${IMAGE_NAME} || true && \
                     docker rm ${IMAGE_NAME} || true && \
                     docker run -d -p ${VM2_PORT}:4444 --name ${IMAGE_NAME} ${IMAGE_URI}"

                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
            withKubeConfig(serverUrl: env.KUBE_SERVER, credentialsId: env.KUBE_CRED) {
                    sh "kubectl auth can-i create pods -n ${K8S_NAMESPACE}"
                    sh "kubectl delete pod ${POD} -n ${K8S_NAMESPACE} --ignore-not-found --wait"
                    sh "kubectl apply -f k8s/pod.yaml -n ${K8S_NAMESPACE}"
                    sh "kubectl apply -f k8s/service.yaml -n ${K8S_NAMESPACE}"
                    sh "kubectl wait --for=condition=Ready pod/${POD} -n ${K8S_NAMESPACE} --timeout=90s"
                    sh "kubectl get pod ${POD} -n ${K8S_NAMESPACE} -o wide"
                    sh "kubectl get service ${POD} -n ${K8S_NAMESPACE} -o wide"
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
