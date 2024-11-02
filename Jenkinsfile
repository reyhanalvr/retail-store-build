pipeline {
    agent any

    environment {
        IMAGE_TAG = "${IMAGE}:${BUILD_NUMBER}"
        DIR = "/home/alvaro/retail-store-build"
    }

    stages {
        stage('Check for UI Changes') {
            steps {
                script {
                    hasUiChanges = sh(
                        script: "git diff --quiet HEAD~1 HEAD -- src/ui || echo 'has_changes'",
                        returnStatus: true
                    ) == 0
                }
            }
        }

        stage('Git Pull on App Servers') {
            when {
                expression { return hasUiChanges }
            }
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                            cd /home/alvaro/retail-store-build &&
                            git fetch origin &&
                            git checkout master &&
                            git pull origin master
                        '
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { return hasUiChanges }
            }
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                            cd ${DIR} &&
                            docker buildx build -t ${IMAGE_TAG} src/ui
                        '
                        """
                    }
                }
            }
        }

        stage('Basic Security Checks') {
            when {
                expression { return hasUiChanges }
            }
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                            # Display Docker Image Layers
                            echo "Displaying Docker Image Layers..." &&
                            docker history ${IMAGE_TAG} &&
                            
                            # Inspect Docker Image Metadata
                            echo "Inspecting Docker Image Metadata..." &&
                            docker inspect ${IMAGE_TAG} | grep -E "User|ExposedPorts|Env" &&
                            
                            # Create temporary container, export and check library versions
                            echo "Checking for known vulnerabilities in critical libraries..." &&
                            CONTAINER_ID=\$(docker create ${IMAGE_TAG}) &&
                            docker export \${CONTAINER_ID} | tar -tvf - | grep -E "libarchive|openssl|curl" &&
                            docker rm \${CONTAINER_ID}
                        '
                        """
                    }
                }
            }
        }

        stage('Docker Registry Login and Push') {
            when {
                expression { return hasUiChanges }
            }
            steps {
                sshagent(credentials: ['ssh-build-server']) {
                    withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        script {
                            sh """
                            ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                                echo ${DOCKER_PASS} | docker login ${DOCKER_REGISTRY} -u ${DOCKER_USER} --password-stdin &&
                                docker push ${IMAGE_TAG}
                            '
                            """
                        }
                    }
                }
            }
        }
        
        stage('Push Docker Image to Registry') {
            when {
                expression { return hasUiChanges }
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

        stage('Change Version for Deployment') {
            when {
                expression { return hasUiChanges }
            }
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_BUILD_SERVER} '
                            cd /home/alvaro/retail-store-deploy/services/ui/deployment &&
                            sed -i "s|image:.*|image: ${IMAGE_TAG}|" ui-deployment.yaml &&
                            git add ui-deployment.yaml &&
                            git commit -m "Update image tag to ${IMAGE_TAG} for Argo CD deployment" &&
                            git push origin master
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
