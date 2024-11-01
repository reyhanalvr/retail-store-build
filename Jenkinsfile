pipeline {
    agent any

    environment {
        IMAGE_TAG = "${IMAGE}:${BUILD_NUMBER}"
    }

    stages {
        stage('Git Pull on App Servers') {
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                            cd ${DIR} &&
                            git checkout ${GIT_BRANCH} &&
                            git pull origin ${GIT_BRANCH}
                        '
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            when {
                changeset "src/ui/**"
            }
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                            cd ${DIR} &&
                            docker build -t ${IMAGE_TAG} src/ui
                        '
                        """
                    }
                }
            }
        }

        stage('Scan Docker Image with Trivy') {
            when {
                changeset "src/ui/**"
            }
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                            trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_TAG}
                        '
                        """
                    }
                }
            }
        }

        stage('Docker Registry Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh """
                        docker login ${DOCKER_REGISTRY} -u ${DOCKER_USER} -p ${DOCKER_PASS}
                        """
                    }
                }
            }
        }

        stage('Push Docker Image to Registry') {
            when {
                changeset "src/ui/**"
            }
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                            docker push ${IMAGE_TAG}
                        '
                        """
                    }
                }
            }
        }

        stage('Deploy on Kubernetes') {
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_DEPLOY_SERVER} '
                            sed -i "s|image: .*\$|image: ${IMAGE_TAG}|g" ${DEPLOYMENT_FILE} &&
                            kubectl apply -f ${DEPLOYMENT_FILE}
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
