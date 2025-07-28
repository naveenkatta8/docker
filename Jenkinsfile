pipeline {
    agent any

    environment {
        IMAGE_NAME = "us-central1-docker.pkg.dev/learn-465307/images/jenkins"
        IMAGE_TAG = "new-${BUILD_NUMBER}"
        SERVICE_ACCOUNT_KEY = credentials('GCR') // File credential
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

        stage('DockerImageBuild on Remote') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'vamij5',
                        transfers: [
                            sshTransfer(
                                sourceFiles: '**/*',
                                removePrefix: '',
                                remoteDirectory: 'docker',
                                execCommand: """
                                    cd docker &&
                                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                                """,
                                execTimeout: 120000
                            )
                        ],
                        usePromotionTimestamp: false,
                        verbose: true
                    )
                ])
            }
        }

        stage('Auth to Artifact Registry') {
            steps {
                sh '''
                    gcloud auth activate-service-account --key-file=$SERVICE_ACCOUNT_KEY
                    gcloud auth configure-docker us-central1-docker.pkg.dev --quiet
                '''
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'vamij5',
                        transfers: [
                            sshTransfer(
                                sourceFiles: '',
                                remoteDirectory: 'docker',
                                execCommand: "docker push ${IMAGE_NAME}:${IMAGE_TAG}",
                                execTimeout: 120000
                            )
                        ],
                        usePromotionTimestamp: false,
                        verbose: true
                    )
                ])
            }
        }
    }
}
