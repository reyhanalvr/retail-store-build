pipeline {
    agent any

    environment {
        GIT_BRANCH = 'master'
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                script {
                    git branch: "master", url: "https://github.com/reyhanalvr/retail-store-build", credentialsId: "github-credentials"
                }
            }
        }
    }
}
