pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "sujeet2016"
        IMAGE_NAME = "myapp"
    }

    stages {

        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/sujeet2016/myapp-source.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "v${env.BUILD_NUMBER}"
                    sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKERHUB_TOKEN')]) {
                    sh '''
                    echo $DOCKERHUB_TOKEN | docker login -u sujeet2016 --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Update Deployment Repo') {
            steps {
                dir('manifests') {
                    script {
                        git url: 'https://github.com/sujeet2016/myapp-deployment.git', branch: 'main'

                        // FIX 1: Use single quotes so $ works safely
                        sh '''
                        sed -i "s|image: .*|image: sujeet2016/myapp:${IMAGE_TAG}|" k8s/deployment.yaml
                        '''

                        // FIX 2: Escape $ in multiline block OR use env.GITHUB_PAT
                        withCredentials([string(credentialsId: 'github-pat', variable: 'GITHUB_PAT')]) {
                            sh """
                            git config user.email "jenkins@example.com"
                            git config user.name "Jenkins"
                            git add .
                            git commit -m "Update image to ${IMAGE_TAG}" || true
                            git push https://sujeet2016:${env.GITHUB_PAT}@github.com/sujeet2016/myapp-deployment.git
                            """
                        }
                    }
                }
            }
        }
    }
}

