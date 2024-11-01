pipeline {
    agent any

    environment {
        GIT_BRANCH = 'master'
        DIR = '/home/alvaro/retail-store-build' // Sesuaikan path ini
    }

    stages {
        stage('Git Pull on App Servers') {
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no alvaro@103.196.153.76 '
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
                changeset "src/ui/**" // Hanya lanjutkan jika ada perubahan di src/ui
            }
            steps {
                script {
                    sshagent(credentials: ['ssh-build-server']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no alvaro@103.196.153.76 '
                            cd ${DIR} &&
                            docker build -t retail-store-ui:latestt src/ui
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
