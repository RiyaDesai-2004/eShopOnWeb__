pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME  = "/home/riya/.dotnet_cli"
        NUGET_PACKAGES   = "/home/riya/.nuget/packages"
        SOLUTION_FILE    = "eShopOnWeb.sln"
        BUILD_CONFIG     = "Release"
        PUBLISH_DIR      = "publish"
    }

    stages {

        stage('Restore') {
            steps {
                sh 'dotnet restore ${SOLUTION_FILE}'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build ${SOLUTION_FILE} --configuration ${BUILD_CONFIG} --no-restore'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    dotnet test ${SOLUTION_FILE} \
                        --configuration ${BUILD_CONFIG} \
                        --no-build \
                        --logger "trx" \
                        --results-directory ./TestResults
                '''
            }
        }

        stage('Publish') {
            steps {
                sh '''
                    dotnet publish src/Web/Web.csproj \
                        --configuration ${BUILD_CONFIG} \
                        --no-build \
                        --output ${PUBLISH_DIR}
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: "${PUBLISH_DIR}/**/*",
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs above.'
        }
        always {
            cleanWs()
        }
    }
}
