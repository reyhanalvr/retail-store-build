pipeline {
    agent any

    environment {
        GIT_BRANCH = 'master'
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                script {
                    // Checkout code
                    git branch: GIT_BRANCH, url: "https://github.com/reyhanalvr/retail-store-build", credentialsId: "github-credentials"
                    
                    // Print the current directory
                    sh "pwd"
                    
                    // Pull latest changes to ensure code is up-to-date
                    sh "git pull origin ${GIT_BRANCH}"
                }
            }
        }
    }
}
