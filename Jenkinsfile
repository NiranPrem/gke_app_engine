pipeline {
    agent any
    environment {
        PROJECT_ID    = "useful-variety-470306-n5"
        GCP_KEY       = credentials('GCP_CREDS') // Your service account key in Jenkins
        APP_FOLDER    = "Deploy_App_In_AppEngine/my-first-service"
        VERSION       = "v${env.BUILD_ID}"       // Auto version per build
    }
    stages {

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
                script {
                    echo "Deploying default service from ${APP_FOLDER} with version ${VERSION}..."
                    sh "gcloud app deploy ${APP_FOLDER}/app.yaml --version=${VERSION} --quiet"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo "App deployed. Access your app at:"
                    sh "gcloud app browse --no-launch-browser"
                }
            }
        }
    }
}
