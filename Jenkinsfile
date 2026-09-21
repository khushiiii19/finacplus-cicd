pipeline {
    agent any

    /*
     * Parameters make the pipeline reusable across
     * different applications and Kubernetes environments.
     */
    parameters {
        string(
            name: 'IMAGE_REPO',
            defaultValue: 'khushiiii19/finacplus-cicd',
            description: 'Docker Hub repository for the application image'
        )

        string(
            name: 'K8S_DEPLOYMENT',
            defaultValue: 'finacplus-app',
            description: 'Kubernetes Deployment name'
        )

        string(
            name: 'K8S_CONTAINER',
            defaultValue: 'finacplus-app',
            description: 'Container name inside the Kubernetes Deployment'
        )

        string(
            name: 'K8S_NAMESPACE',
            defaultValue: 'default',
            description: 'Kubernetes namespace'
        )

        string(
            name: 'K8S_CONTEXT',
            defaultValue: 'docker-desktop',
            description: 'Kubernetes context to deploy to'
        )
    }

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
        buildDiscarder(
            logRotator(
                numToKeepStr: '10'
            )
        )
    }

    environment {
        DOCKER_CREDENTIALS = 'dockerhub-creds'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Running application validation...'

                sh '''
                    set -e
                    python3 --version
                    python3 -m py_compile app/app.py
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image: ${params.IMAGE_REPO}:${BUILD_NUMBER}"

                sh '''
                    set -e
                    docker build \
                        -t "${IMAGE_REPO}:${BUILD_NUMBER}" \
                        .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set -e

                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push "${IMAGE_REPO}:${BUILD_NUMBER}"
                    '''
                }
            }

            post {
                always {
                    sh 'docker logout || true'
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                echo "Deploying ${params.IMAGE_REPO}:${BUILD_NUMBER} to Kubernetes..."

                sh '''
                    set -e

                    kubectl --context="${K8S_CONTEXT}" \
                        --namespace="${K8S_NAMESPACE}" \
                        set image deployment/"${K8S_DEPLOYMENT}" \
                        "${K8S_CONTAINER}"="${IMAGE_REPO}:${BUILD_NUMBER}"

                    kubectl --context="${K8S_CONTEXT}" \
                        --namespace="${K8S_NAMESPACE}" \
                        rollout status deployment/"${K8S_DEPLOYMENT}" \
                        --timeout=120s
                '''
            }
        }

        stage('Deployment Verification') {
            steps {
                echo 'Verifying Kubernetes deployment...'

                sh '''
                    set -e

                    echo "Deployment status:"
                    kubectl --context="${K8S_CONTEXT}" \
                        --namespace="${K8S_NAMESPACE}" \
                        get deployment "${K8S_DEPLOYMENT}"

                    echo "Pod status:"
                    kubectl --context="${K8S_CONTEXT}" \
                        --namespace="${K8S_NAMESPACE}" \
                        get pods \
                        -l app="${K8S_CONTAINER}"
                '''
            }
        }
    }

    post {
        success {
            echo """
            Pipeline completed successfully.

            Docker Image:
            ${params.IMAGE_REPO}:${BUILD_NUMBER}

            Kubernetes Deployment:
            ${params.K8S_DEPLOYMENT}

            Namespace:
            ${params.K8S_NAMESPACE}
            """
        }

        failure {
            echo """
            Pipeline failed.

            Check the failed stage and its console output
            to identify the root cause.
            """
        }

        always {
            echo "Build #${BUILD_NUMBER} completed."
        }
    }
}
