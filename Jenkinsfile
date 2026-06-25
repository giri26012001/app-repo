pipeline {
    agent any
    
    environment {
        IMAGE_NAME    = 'sample-app'
        IMAGE_TAG     = "${BUILD_NUMBER}"
        MANIFEST_REPO = 'k8s-manifest-repo'
    }
    
    stages {
        stage('Build & Push Docker Image') {
            steps {
                // Dynamically binds BOTH the username and password fields from your docker-hub-credentials profile
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USER')]) {
                    script {
                        echo "Building Docker Image for user: ${DOCKER_HUB_USER}..."
                        sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                        
                        echo "Logging into Docker Hub..."
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USER} --password-stdin"
                        
                        echo "Pushing Image..."
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
        
        stage('Update K8s Manifests') {
            steps {
                // Dynamically binds BOTH the username and password fields from your github-credentials profile
                withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh """
                        # Configure local Git identity
                        git config --global user.email "jenkins@local.internal"
                        git config --global user.name "Jenkins Automation"
                        
                        # Clone the infrastructure repository cleanly using dynamic environment variables
                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/${MANIFEST_REPO}.git
                        cd ${MANIFEST_REPO}
                        
                        # Use double quotes for sed to safely handle the injected environment variables
                        sed -i "s|image: .*|image: docker.io/${GIT_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}|g" templates/deployment.yaml
                        
                        # Commit and push the change back to the main branch
                        git add templates/deployment.yaml
                        git commit -m "Automated build update: Image tag ${IMAGE_TAG} [skip ci]"
                        git push origin main
                    """
                }
            }
        }
    }
}