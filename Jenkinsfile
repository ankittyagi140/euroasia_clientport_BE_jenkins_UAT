/**
 * Client Portal BE — UAT / dev CI
 *
 * Jenkins job  : euroasia_clientport_BE_jenkins_UAT (this repo)
 * Script Path  : Jenkinsfile
 *
 * Checks out the application repo, tests, builds, pushes GHCR, then
 * triggers the platform job with APPLICATION=client-portal-be.
 *
 * App repo     : https://github.com/ankittyagi140/euroasia_clientportal_BE.git
 * Image        : ghcr.io/euroasiasci/client-portal-be
 * Credentials  : github-pat, ghcr-token
 */

pipeline {

    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 45, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '30'))
        skipDefaultCheckout(true)
    }

    environment {

        // ==========================================================
        // Application
        // ==========================================================

        APP_NAME = "client-portal-be"

        REPO_URL = "https://github.com/ankittyagi140/euroasia_clientportal_BE.git"

        GIT_BRANCH = "dev"

        // ==========================================================
        // Docker
        // ==========================================================

        DOCKER_BUILDKIT = "1"

        DOCKERFILE = "Dockerfile"

        REGISTRY = "ghcr.io/euroasiasci"

        IMAGE = "${REGISTRY}/${APP_NAME}"

        IMAGE_TAG = "${BUILD_NUMBER}"

        ENVIRONMENT = "dev"

        // ==========================================================
        // Platform CD Job
        // ==========================================================

        PLATFORM_JOB = "EUROASIA_SCI_CLIENT_PORTAL/Euroasia_Inspection/Euroasia_inspection_Infra/Euroasia_inspection_Infra"

    }

    stages {

        stage('Checkout') {

            steps {

                git(

                    branch: GIT_BRANCH,

                    credentialsId: "github-pat",

                    url: REPO_URL

                )

                script {

                    env.GIT_SHA = sh(

                        script: "git rev-parse --short HEAD",

                        returnStdout: true

                    ).trim()

                }

            }

        }

        stage('Lint & Unit Test') {

            steps {

                sh '''

                set -eux

                CI_CONTAINER="client-portal-be-ci-${BUILD_NUMBER}"

                docker rm -f "${CI_CONTAINER}" >/dev/null 2>&1 || true

                cleanup() { docker rm -f "${CI_CONTAINER}" >/dev/null 2>&1 || true; }
                trap cleanup EXIT

                docker run \
                    -d \
                    --name "${CI_CONTAINER}" \
                    -w /workspace \
                    ghcr.io/astral-sh/uv:python3.13-bookworm-slim \
                    sleep infinity

                docker cp "$PWD/." "${CI_CONTAINER}:/workspace"

                docker exec \
                    "${CI_CONTAINER}" \
                    bash /workspace/scripts/ci-lint-test.sh

                docker cp \
                    "${CI_CONTAINER}:/workspace/coverage.xml" \
                    coverage.xml || true

                docker cp \
                    "${CI_CONTAINER}:/workspace/htmlcov" \
                    htmlcov || true

                '''

            }

        }

        stage('Build Docker Image') {

            steps {

                sh '''

                set -eux

                docker build \
                    --pull \
                    --build-arg BUILDKIT_INLINE_CACHE=1 \
                    --label org.opencontainers.image.version=${BUILD_NUMBER} \
                    --label org.opencontainers.image.revision=${GIT_SHA} \
                    --label org.opencontainers.image.source=${REPO_URL} \
                    --label org.opencontainers.image.title=client-portal-be \
                    -f ${DOCKERFILE} \
                    -t ${IMAGE}:${IMAGE_TAG} \
                    -t ${IMAGE}:${ENVIRONMENT} \
                    .

                '''

            }

        }

        stage('Login GHCR') {

            steps {

                withCredentials([

                    usernamePassword(

                        credentialsId: 'ghcr-token',

                        usernameVariable: 'GHCR_USER',

                        passwordVariable: 'GHCR_TOKEN'

                    )

                ]) {

                    sh '''

                    echo "${GHCR_TOKEN}" | docker login ghcr.io \
                        --username "${GHCR_USER}" \
                        --password-stdin

                    '''

                }

            }

        }

        stage('Push Docker Image') {

            steps {

                sh '''

                set -eux

                docker push ${IMAGE}:${IMAGE_TAG}

                docker push ${IMAGE}:${ENVIRONMENT}

                '''

            }

        }

        stage('Trigger Platform Deployment') {

            steps {

                script {

                    echo "========================================="

                    echo "Triggering Platform Deployment..."

                    echo "Application : ${APP_NAME}"

                    echo "Environment : ${ENVIRONMENT}"

                    echo "Image Tag : ${IMAGE_TAG}"

                    echo "========================================="

                    build(

                        job: PLATFORM_JOB,

                        wait: true,

                        propagate: true,

                        parameters: [

                            string(

                                name: 'APPLICATION',

                                value: APP_NAME

                            ),

                            string(

                                name: 'ENVIRONMENT',

                                value: ENVIRONMENT

                            ),

                            string(

                                name: 'IMAGE_TAG',

                                value: IMAGE_TAG

                            ),

                            booleanParam(

                                name: 'DRY_RUN',

                                value: false

                            ),

                            booleanParam(

                                name: 'SKIP_HEALTH_CHECK',

                                value: false

                            )

                        ]

                    )

                }

            }

        }

    }

    post {

        success {

            archiveArtifacts(

                artifacts: 'coverage.xml,htmlcov/**',

                allowEmptyArchive: true

            )

            echo '''

=========================================

Client Portal BE CI Successful

Docker image published

Platform deployment completed

=========================================

'''

        }

        failure {

            echo '''

=========================================

Client Portal BE CI Failed

Check Platform Deployment logs if the
failure occurred after the image push.

=========================================

'''

        }

        always {

            sh '''

            docker logout ghcr.io || true

            '''

            cleanWs()

        }

    }

}
