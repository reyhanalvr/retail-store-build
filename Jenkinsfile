pipeline {
    agent any

    environment {
        GIT_BRANCH = 'master'
        REGISTRY_URL = 'registry.alvaro.studentdumbways.my.id'
        REPO_NAME = 'retail-store-sample'
        VERSION_FILE = 'VERSION'
        DOCKER_CREDENTIALS_ID = 'docker-registry-credentials' // Update with Docker credentials ID in Jenkins
        GITHUB_CREDENTIALS_ID = 'github-credentials' // Update with GitHub credentials ID in Jenkins
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

        stage('Check Docker') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Update Version') {
            steps {
                script {
                    // Read the current version from the VERSION file
                    def currentVersion = readFile(VERSION_FILE).trim()
                    echo "Current version: ${currentVersion}"

                    // Increment the patch version
                    def versionParts = currentVersion.split('\\.')
                    versionParts[2] = (versionParts[2].toInteger() + 1).toString()
                    def newVersion = versionParts.join('.')
                    echo "New version: ${newVersion}"

                    // Update the VERSION file with the new version
                    writeFile file: VERSION_FILE, text: newVersion

                    // Set the Docker image tag with the new version
                    env.UI_IMAGE = "${REGISTRY_URL}/${REPO_NAME}/ui:${newVersion}"
                    
                    // Commit and push the updated VERSION file
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, passwordVariable: 'GITHUB_PASSWORD', usernameVariable: 'GITHUB_USERNAME')]) {
                        sh """
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git add ${VERSION_FILE}
                        git commit -m "Update version to ${newVersion}" || echo "No changes to commit"
                        git remote set-url origin https://${GITHUB_USERNAME}:${GITHUB_PASSWORD}@github.com/reyhanalvr/retail-store-build.git
                        git push origin ${GIT_BRANCH}
                        """
                    }
                }
            }
        }

        stage('Build UI Image') {
            when {
                changeset "src/ui/**"  // Only run this stage if there are changes in src/ui
            }
            steps {
                script {
                    echo "Changes detected in UI service. Building UI Docker image."
                    
                    // Build the Docker image
                    sh "docker build -t ${env.UI_IMAGE} src/ui"
                    
                    // Login to Docker registry
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                        echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin ${REGISTRY_URL}
                        """
                    }
                    
                    // Push the Docker image
                    sh "docker push ${env.UI_IMAGE}"
                }
            }
        }
    }
}
