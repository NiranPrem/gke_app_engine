pipeline {
    agent any
    environment {
        PROJECT_ID = "useful-variety-470306-n5"
        GCP_KEY    = credentials('GCP_CREDS')  // Jenkins secret JSON key
        APP_FOLDER = "Deploy_App_In_AppEngine/new"  // Path to app.yaml
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

        stage('Cleanup Old Versions') {
            steps {
                script {
                    // Keep only the latest 3 versions
                    def keep = 3
                    def versions = sh(
                        script: "gcloud app versions list --service=default --sort-by=~version.id --format='value(version.id)'",
                        returnStdout: true
                    ).trim().split("\\n")

                    if (versions.size() > keep) {
                        def toDelete = versions[keep..-1].join(" ")
                        echo "Deleting old versions: ${toDelete}"
                        sh "gcloud app versions delete ${toDelete} --service=default --quiet"
                    } else {
                        echo "No old versions to delete."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
