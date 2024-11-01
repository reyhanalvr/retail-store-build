pipeline {
    agent any

    environment {
        IMAGE_TAG = "${IMAGE}:${BUILD_NUMBER}"
        DIR = "/home/alvaro/retail-store-build"
    }

    stages {
        stage('Git Pull on App Servers') {
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

        // Multi Stage Pipeline -> retail-store-deployment
        // stage('Trigger Deployment Pipeline') {
        //     steps {
        //         script {
        //             // Trigger Jenkins job for deployment
        //             build job: 'retail-store-deployment/deployment-pipeline', parameters: [string(name: 'IMAGE_TAG', value: "${IMAGE_TAG}")]
        //         }
        //     }
        // }

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
                            docker buildx build -t ${IMAGE_TAG} src/ui
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

        stage('Docker Registry Login and Push') {
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
