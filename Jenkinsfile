pipeline {
    agent any

    environment {
        IMAGE_NAME = "backend-app" // Just image name, no URL here
        IMAGE_TAG = "v${BUILD_NUMBER}"
        FULL_IMAGE = "54.81.112.24:9000/${IMAGE_NAME}:${IMAGE_TAG}"

        APP_REPO_URL = "https://github.com/farhanfist10/applicationcode.git"
        MANIFEST_REPO_URL = "https://github.com/farhanfist10/manifests.git"
        MANIFEST_REPO_BRANCH = "main"
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: "${APP_REPO_URL}", branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}") // Build only, locally
                }
            }
        }

        stage('Push Docker Image to Nexus') {
            environment {
                DOCKER_CREDS = credentials('nexus-docker-credentials') // ID from Jenkins credentials
            }
            steps {
                script {
                    docker.withRegistry('http://54.81.112.24:9000', 'nexus-docker-credentials') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Clone Manifest Repo and Update Image Tag') {
            environment {
                TOKEN = credentials('github-token')
            }
            steps {
                sh """
                rm -rf manifests
                git clone --branch $MANIFEST_REPO_BRANCH $MANIFEST_REPO_URL
                cd manifests
                git config user.name "jenkins"
                git config user.email "jenkins@ci.local"

                git checkout -b update-image-$BUILD_NUMBER
                sed -i "s|image: .*|image: 13.58.246.191:5000/${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml

                git add deployment.yaml
                git commit -m "Update image to 54.81.112.24:9000/${IMAGE_NAME}:${IMAGE_TAG}"
                git push https://$TOKEN@github.com/farhanfist10/manifests.git update-image-$BUILD_NUMBER
                """
            }
        }
    }
}
