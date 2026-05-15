pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME  = "/home/riya/.dotnet_cli"
        NUGET_PACKAGES   = "/home/riya/.nuget/packages"
        SOLUTION_FILE    = "eShopOnWeb.sln"
        BUILD_CONFIG     = "Release"
        PUBLISH_DIR      = "publish"
        DC_HOME          = "/opt/dependency-check/bin"
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

        // ── OWASP Dependency Check ────────────────────────────────
        stage('OWASP Dependency Check') {
            steps {
                sh '''
                    mkdir -p dependency-check-report

                    ${DC_HOME}/dependency-check.sh \
                        --scan ./ \
                        --format HTML \
                        --format XML \
                        --out ./dependency-check-report \
                        --project "eShopOnWeb" \
                        --prettyPrint \
                        --enableExperimental
                '''
            }
            post {
                always {
                    // Publish XML report to Jenkins dashboard
                    dependencyCheckPublisher(
                        pattern: 'dependency-check-report/dependency-check-report.xml',
                        failedTotalCritical: 0,
                        failedTotalHigh: 5,
                        unstableTotalHigh: 3
                    )
                    // Also archive the HTML report
                    archiveArtifacts artifacts: 'dependency-check-report/**/*',
                                     allowEmptyArchive: true
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
