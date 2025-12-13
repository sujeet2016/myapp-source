pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "sujeet0508"
        IMAGE_NAME = "myapp"
        IMAGE_TAG = ""
    }

    stages {

        stage('Checkout Source') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sujeet2016/myapp-source.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "v${env.BUILD_NUMBER}"
                    sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${env.IMAGE_TAG} ."
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKERHUB_TOKEN')]) {
                    sh '''
                    echo $DOCKERHUB_TOKEN | docker login -u sujeet0508 --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${env.IMAGE_TAG}"
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

