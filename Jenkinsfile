pipeline {
    agent any

    environment {
        GIT_BRANCH = 'master'
        REGISTRY_URL = 'registry.alvaro.studentdumbways.my.id'
        REPO_NAME = 'retail-store-sample'
        VERSION_FILE = 'VERSION'
        DOCKER_CREDENTIALS_ID = 'docker-registry-credentials' // Update with your Docker registry credentials ID
        GITHUB_CREDENTIALS_ID = 'github-credentials' // Update with your GitHub credentials ID
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                script {
                    // Checkout code from GitHub
                    git branch: GIT_BRANCH, url: "https://github.com/reyhanalvr/retail-store-build", credentialsId: GITHUB_CREDENTIALS_ID
                }
            }
        }

        stage('Update Version') {
            steps {
                script {
                    // Read current version from the VERSION file
                    def currentVersion = readFile(VERSION_FILE).trim()
                    echo "Current version: ${currentVersion}"

                    // Increment version
                    def versionParts = currentVersion.split('\\.')
                    versionParts[2] = (versionParts[2].toInteger() + 1).toString() // Increment patch version
                    def newVersion = versionParts.join('.')
                    echo "New version: ${newVersion}"

                    // Update the VERSION file with new version
                    writeFile file: VERSION_FILE, text: newVersion

                    // Set Docker image tag using new version
                    env.UI_IMAGE = "${REGISTRY_URL}/${REPO_NAME}/ui:${newVersion}"
                    
                    // Commit and push updated VERSION file
                    sh """
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins"
                    git add ${VERSION_FILE}
                    git commit -m "Update version to ${newVersion}"
                    git push origin ${GIT_BRANCH}
                    """
                }
            }
        }

        stage('Build UI Image') {
            when {
                changeset "src/ui/**"  // Only trigger this stage if there are changes in src/ui
            }
            steps {
                script {
                    echo "Changes detected in UI service. Building UI Docker image."
                    
                    // Build Docker image
                    sh "docker build -t ${env.UI_IMAGE} src/ui"
                    
                    // Login to Docker registry
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                        echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin ${REGISTRY_URL}
                        """
                    }
                    
                    // Push Docker image
                    sh "docker push ${env.UI_IMAGE}"
                }
            }
        }

    }
}
