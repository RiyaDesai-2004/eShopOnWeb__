pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME = "/tmp/dotnet_cli_home"
        SOLUTION_FILE   = "eShopOnWeb.sln"
        BUILD_CONFIG    = "Release"
        PUBLISH_DIR     = "publish"
    }

    stages {

        // ── 1. Checkout ──────────────────────────────────────────
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/RiyaDesai-2004/eShopOnWeb__.git'
            }
        }

        // ── 2. Restore NuGet packages ────────────────────────────
        stage('Restore') {
            steps {
                sh 'dotnet restore ${SOLUTION_FILE}'
            }
        }

        // ── 3. Build ─────────────────────────────────────────────
        stage('Build') {
            steps {
                sh 'dotnet build ${SOLUTION_FILE} --configuration ${BUILD_CONFIG} --no-restore'
            }
        }

        // ── 4. Run Unit Tests ────────────────────────────────────
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
                    // Publish test results
                    junit '**/TestResults/*.trx'
                }
            }
        }

        // ── 5. Publish ───────────────────────────────────────────
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

        // ── 6. Archive Artifacts ─────────────────────────────────
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: "${PUBLISH_DIR}/**/*",
                                 fingerprint: true
            }
        }

        // ── 7. Docker Build & Push (Optional) ───────────────────
        stage('Docker Build') {
            when {
                branch 'main'   // Only on main branch
            }
            steps {
                script {
                    def image = docker.build("eshop-web:${env.BUILD_NUMBER}")
                    // To push to Docker Hub / registry, uncomment:
                    // docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                    //     image.push()
                    //     image.push('latest')
                    // }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs above.'
            // emailext subject: "Build Failed: ${env.JOB_NAME}",
            //          body: "Check Jenkins: ${env.BUILD_URL}",
            //          to: 'your@email.com'
        }
        always {
            cleanWs()   // Clean workspace after every run
        }
    }
}
