pipeline {
    agent any

    environment {
        IMAGE_NAME = "us-central1-docker.pkg.dev/learn-465307/images/jenkins"
        IMAGE_TAG = "new-${BUILD_NUMBER}"
        SERVICE_ACCOUNT_KEY = credentials('GCR') // ID of your secret file credential in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/keyspaceits/docker.git']]
                )
            }
        }

        stage('DockerImageBuild') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Auth to Artifact Registry') {
            steps {
                sh '''
                echo $SERVICE_ACCOUNT_KEY >sa-key.json
                gcloud auth activate-service-account --key-file=sa-key.json
                gcloud auth configure-docker us-central1-docker.pkg.dev --quiet
                '''
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                sh """
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }}

