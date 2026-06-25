pipeline {
    agent any
        tools {
            dockerTool 'docker'
        }
    
    environment {
        DOCKER_HUB_USER = credentials('dockerhub_username')
        IMAGE_NAME      = 'sample-app'
        IMAGE_TAG       = "${BUILD_NUMBER}" // Uses Jenkins build number as image tag
        GITHUB_USER     = credentials('github_username')
        MANIFEST_REPO   = 'k8s-manifest-repo'
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                // Uses the credential ID we created earlier in Jenkins
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        
        stage('Update K8s Manifests') {
            steps {
                // Uses the GitHub Classic Token credential we created earlier
                withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh """
                        # Configure local Git identity inside container
                        git config --global user.email "jenkins@local.internal"
                        git config --global user.name "Jenkins Automation"
                        
                        # Clone the infrastructure repository cleanly
                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GITHUB_USER}/${MANIFEST_REPO}.git
                        cd ${MANIFEST_REPO}
                        
                        # Use sed to update the image tag dynamically in your deployment configuration
                        sed -i 's|image: .*|image: docker.io/${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|g' templates/deployment.yaml
                        
                        # Commit the change back to the main branch
                        git add templates/deployment.yaml
                        git commit -m "Automated build update: Image tag ${IMAGE_TAG} [skip ci]"
                        git push origin main
                    """
                }
            }
        }
    }
}