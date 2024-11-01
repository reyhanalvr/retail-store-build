pipeline {
    agent any

    environment {
        GIT_BRANCH = 'master'
        REGISTRY_URL = 'registry.alvaro.studentdumbways.my.id'
        UI_IMAGE = "${REGISTRY_URL}/retail-store-sample/ui:latest"
        DOCKER_USERNAME = 'admin' // Ganti dengan username registry
        DOCKER_PASSWORD = 'Thebe@tles45' // Ganti dengan password registry
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

        stage('Build UI Image') {
            when {
                changeset "src/ui/**"  // Runs this stage only if there are changes in src/ui directory
            }
            steps {
                script {
                    echo "Changes detected in UI service. Building UI Docker image."

                    // Build Docker image
                    sh """
                        cd src/ui
                        docker build --no-cache -t ${UI_IMAGE} .
                    """

                    // Docker login
                    sh """
                        echo ${DOCKER_PASSWORD} | docker login ${REGISTRY_URL} -u ${DOCKER_USERNAME} --password-stdin
                    """

                    // Push Docker image
                    sh """
                        docker push ${UI_IMAGE}
                    """
                }
            }
        }
    }
}
