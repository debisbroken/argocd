pipeline {
    agent any
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag')
    }
    environment {
        GIT_REPO = "https://github.com/Anonymous-2009/argocd.git"
        GIT_BRANCH = "main"
        DEPLOYMENT_FILE = "dep.yml"
        DOCKER_IMAGE = "anonymous2009/my-express-app"
    }
    stages {
        stage('Checkout GitOps Repo') {
            steps {
                script {
                    checkout([$class: 'GitSCM', 
                        branches: [[name: "*/${GIT_BRANCH}"]], 
                        userRemoteConfigs: [[url: GIT_REPO, credentialsId: 'git-credentials']],
                        extensions: [[$class: 'LocalBranch', localBranch: "${GIT_BRANCH}"]]
                    ])
                }
            }
        }
        stage('Update Image Tag') {
            steps {
                script {
                    def filePath = DEPLOYMENT_FILE
                    def oldTag = sh(script: "grep -o '${DOCKER_IMAGE}:.*' ${filePath} | cut -d':' -f2", returnStdout: true).trim()
                    
                    if (oldTag) {
                        sh "sed -i 's|${DOCKER_IMAGE}:${oldTag}|${DOCKER_IMAGE}:${IMAGE_TAG}|g' ${filePath}"
                    } else {
                        error "Could not find image tag in ${DEPLOYMENT_FILE}"
                    }
                    echo "Updated image tag to: ${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }
        stage('Commit & Push Changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    script {
                        sh """
                        git config --global user.email "jenkins@example.com"
                        git config --global user.name "Jenkins"
                        git add ${DEPLOYMENT_FILE}
                        git commit -m "Update image to ${DOCKER_IMAGE}:${IMAGE_TAG}" || echo "No changes to commit"
                        git push https://\${GIT_USER}:\${GIT_PASS}@github.com/Anonymous-2009/argocd.git ${GIT_BRANCH}
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Build & Push successful."
            emailext subject: "Jenkins Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "Build and push completed successfully. Docker Image: ${DOCKER_IMAGE}:${params.IMAGE_TAG}",
                     to: 'krishnabag751@gmail.com'
        }
        failure {
            echo "Build failed!"
            emailext subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "Jenkins build has failed. Check logs for details.",
                     to: 'krishnabag751@gmail.com'
        }
    }
}