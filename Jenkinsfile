pipeline {
    agent any

    environment {
        GIT_BRANCH = 'master'
        REGISTRY_URL = 'registry.alvaro.studentdumbways.my.id'
        REPO_NAME = 'retail-store-sample'
        VERSION_FILE = 'VERSION'
        DOCKER_CREDENTIALS_ID = 'docker-registry-credentials'
        GITHUB_CREDENTIALS_ID = 'github-credentials'
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                dir('repo') { // Use a consistent directory for Git operations
                    checkout scm: [
                        $class: 'GitSCM', 
                        branches: [[name: GIT_BRANCH]], 
                        userRemoteConfigs: [[url: "https://github.com/reyhanalvr/retail-store-build", credentialsId: GITHUB_CREDENTIALS_ID]]
                    ]
                }
            }
        }

        stage('Update Version and Push') {
            steps {
                dir('repo') {
                    script {
                        def currentVersion = readFile(VERSION_FILE).trim()
                        def versionParts = currentVersion.split('\\.')
                        versionParts[2] = (versionParts[2].toInteger() + 1).toString()
                        def newVersion = versionParts.join('.')
                        writeFile file: VERSION_FILE, text: newVersion
                        env.UI_IMAGE = "${REGISTRY_URL}/${REPO_NAME}/ui:${newVersion}"

                        withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, passwordVariable: 'GITHUB_PASSWORD', usernameVariable: 'GITHUB_USERNAME')]) {
                            sh """
                            git config user.email "jenkins@example.com"
                            git config user.name "Jenkins"
                            git add ${VERSION_FILE}
                            git commit -m "Update version to ${newVersion}" || true
                            git push https://${GITHUB_USERNAME}:${GITHUB_PASSWORD}@github.com/reyhanalvr/retail-store-build.git ${GIT_BRANCH}
                            """
                        }
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            when {
                changeset "src/ui/**"
            }
            steps {
                dir('repo') {
                    script {
                        sh "docker build -t ${env.UI_IMAGE} src/ui"
                        withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                            sh """
                            echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin ${REGISTRY_URL}
                            docker push ${env.UI_IMAGE}
                            """
                        }
                    }
                }
            }
        }
    }
}
