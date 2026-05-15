pipeline {
    agent any

    environment {
        // ✅ FIXED: Point away from /tmp to home directory
        DOTNET_CLI_HOME  = "/home/riya/.dotnet_cli"
        NUGET_PACKAGES   = "/home/riya/.nuget/packages"
        SOLUTION_FILE    = "eShopOnWeb.sln"
        BUILD_CONFIG     = "Release"
        PUBLISH_DIR      = "publish"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'main_creds',
                    url: 'https://github.com/RiyaDesai-2004/eShopOnWeb__.git'
            }
        }

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
                        --logger "trx;LogFileName=test-results.trx" \
                        --results-directory ./TestResults
                '''
            }
            post {
                always {
                    junit '**/TestResults/*.trx'
                }
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
