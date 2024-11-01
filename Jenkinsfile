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
    }

    post {
        success {
            echo 'Git pull successfully completed on app server.'
        }
        failure {
            echo 'Git pull failed on app server.'
        }
    }
}
