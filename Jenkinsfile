pipeline {
    agent any

    environment {
        PROJECT_ID    = "useful-variety-470306-n5"
        GCP_KEY       = credentials('GCP_CREDS')
        SERVICE_NAME  = "my-first-service"        // Custom App Engine service
        APP_DIR       = "Deploy_App_In_AppEngine/my-first-service"
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

        stage('Deploy App Engine Service') {
            steps {
                script {
                    // Generate version using timestamp
                    def VERSION_ID = "v${new Date().format('yyyyMMddHHmmss')}"

                    sh """
                        echo "Deploying service $SERVICE_NAME with version $VERSION_ID..."
                        gcloud app deploy ${APP_DIR}/app.yaml \
                            --version=${VERSION_ID} \
                            --quiet
                    """
                }
            }
        }

        stage('Approval Gate: Switch Traffic') {
            steps {
                script {
                    input(
                        id: 'SwitchTrafficApproval',
                        message: "Do you want to send 100% traffic to the new version?",
                        ok: "Switch",
                        submitter: "niranprem"
                    )
                }
            }
        }

        stage('Switch Traffic') {
            steps {
                script {
                    def VERSION_ID = sh(
                        script: "gcloud app versions list --service=${SERVICE_NAME} --sort-by=~LAST_DEPLOYED --format='value(version.id)' | head -n 1",
                        returnStdout: true
                    ).trim()

                    sh """
                        echo "Routing 100% traffic to version ${VERSION_ID}..."
                        gcloud app services set-traffic ${SERVICE_NAME} --splits ${VERSION_ID}=1 --quiet
                    """
                }
            }
        }

        stage('Approval Gate: Cleanup Old Versions') {
            steps {
                script {
                    input(
                        id: 'CleanupOldVersions',
                        message: "Do you want to delete old versions of $SERVICE_NAME now?",
                        ok: "Delete",
                        submitter: "niranprem"
                    )
                }
            }
        }

        stage('Cleanup Old Versions') {
            steps {
                script {
                    // Keep only the latest version, delete the rest
                    sh """
                        gcloud app versions list --service=${SERVICE_NAME} --sort-by=~LAST_DEPLOYED --format='value(version.id)' | tail -n +2 | xargs -r gcloud app versions delete -q --service=${SERVICE_NAME}
                    """
                }
            }
        }

    } // stages
}
