pipeline {
    agent any
    environment {
        PROJECT_ID = "useful-variety-470306-n5"
        GCP_KEY = credentials('GCP_CREDS')
        SERVICE_PATH = "Deploy_App_In_AppEngine/Deploy_App_In_AppEngine/my-first-service"
        VERSION = "v${BUILD_NUMBER}"
    }
    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Authenticate GCP') {
            steps {
                withCredentials([file(credentialsId: 'GCP_CREDS', variable: 'GCP_KEY')]) {
                    sh """
                        echo "Authenticating with GCP..."
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud config set project $PROJECT_ID
                    """
                }
            }
        }

        stage('Deploy App Engine Default Service') {
            steps {
                echo "Deploying default service from ${SERVICE_PATH} with version ${VERSION}..."
                sh """
                    gcloud app deploy ${SERVICE_PATH}/app.yaml --version=${VERSION} --quiet
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Deployment finished. You can view the service at:"
                sh """
                    gcloud app browse
                """
            }
        }
    }
}
