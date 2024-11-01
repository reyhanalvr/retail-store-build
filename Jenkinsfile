pipeline {
    agent any

    environment {
        GIT_BRANCH = 'master'
        REGISTRY_URL = 'registry.alvaro.studentdumbways.my.id'
        REPO_NAME = 'retail-store-sample'
        VERSION_FILE = 'VERSION'
        DOCKER_CREDENTIALS_ID = 'docker-registry-credentials' // Sesuaikan dengan Docker credentials di Jenkins
        GITHUB_CREDENTIALS_ID = 'github-credentials' // Sesuaikan dengan GitHub credentials di Jenkins
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                script {
                    // Checkout code dari GitHub
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
                    // Baca versi saat ini dari file VERSION
                    def currentVersion = readFile(VERSION_FILE).trim()
                    echo "Current version: ${currentVersion}"

                    // Increment versi
                    def versionParts = currentVersion.split('\\.')
                    versionParts[2] = (versionParts[2].toInteger() + 1).toString() // Increment patch version
                    def newVersion = versionParts.join('.')
                    echo "New version: ${newVersion}"

                    // Update file VERSION dengan versi baru
                    writeFile file: VERSION_FILE, text: newVersion

                    // Set tag Docker image dengan versi baru
                    env.UI_IMAGE = "${REGISTRY_URL}/${REPO_NAME}/ui:${newVersion}"
                    
                    // Commit dan push file VERSION yang telah diperbarui
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, passwordVariable: 'GITHUB_PASSWORD', usernameVariable: 'GITHUB_USERNAME')]) {
                        sh """
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git add ${VERSION_FILE}
                        git commit -m "Update version to ${newVersion}"
                        git remote set-url origin https://${GITHUB_USERNAME}:${GITHUB_PASSWORD}@github.com/reyhanalvr/retail-store-build.git
                        git push origin ${GIT_BRANCH}
                        """
                    }
                }
            }
        }

        stage('Build UI Image') {
            when {
                changeset "src/ui/**"  // Hanya jalankan stage ini jika ada perubahan di src/ui
            }
            steps {
                script {
                    echo "Changes detected in UI service. Building UI Docker image."
                    
                    // Build Docker image
                    sh "docker build -t ${env.UI_IMAGE} src/ui"
                    
                    // Login ke Docker registry
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
