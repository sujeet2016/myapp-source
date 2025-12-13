pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "sujeet0508"
        IMAGE_NAME = "myapp"
    }

    stages {

        stage('Checkout Source') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sujeet2016/myapp-source.git'
            }
        }

        stage('Build & Push Image') {
            steps {
                script {
                    def imageTag = "v${env.BUILD_NUMBER}"

                    sh """
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${imageTag} .
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${imageTag}
                    """

                    // export ONLY after creation
                    env.IMAGE_TAG = imageTag
                }
            }
        }

        stage('Update Deployment Repo') {
            steps {
                dir('manifests') {
                    git branch: 'main',
                        url: 'https://github.com/sujeet2016/myapp-deployment.git'

                    sh """
                    sed -i 's|image: .*|image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${env.IMAGE_TAG}|' k8s/deployment.yaml
                    """

                    withCredentials([string(credentialsId: 'github-pat', variable: 'GITHUB_PAT')]) {
                        sh """
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git add k8s/deployment.yaml
                        git commit -m "Update image to ${env.IMAGE_TAG}" || true
                        git push https://sujeet2016:${GITHUB_PAT}@github.com/sujeet2016/myapp-deployment.git
                        """
                    }
                }
            }
        }
    }
}


