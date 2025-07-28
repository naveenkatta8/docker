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
                   sshPublisher(publishers: [sshPublisherDesc(configName: 'vamij5', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''ch docker/
sudo docker.build -t ("${IMAGE_NAME}:${IMAGE_TAG}") .''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'docker', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
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

        }

        stage('Push to Artifact Registry') {
            steps {
                sh """
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

