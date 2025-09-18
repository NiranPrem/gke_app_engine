pipeline {
    agent any
    environment {
        PROJECT_ID    = "useful-variety-470306-n5"
        GCP_KEY       = credentials('GCP_CREDS')
        APP_FOLDER    = "Deploy_App_In_AppEngine/my-first-service" // your current folder
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
                    def version = "v${new Date().format('yyyyMMddHHmmss')}"
                    echo "Deploying default service with version ${version}..."

                    sh """
                        gcloud app deploy ${APP_FOLDER}/app.yaml --version=${version} --quiet
                        gcloud app services set-traffic default --splits ${version}=1.0 --quiet
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    echo "App deployed! Access it at:"
                    gcloud app browse --no-launch-browser
                """
            }
        }

    }
}
