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

        IMAGE_REPO_VALUE = "${params.IMAGE_REPO}"
        K8S_DEPLOYMENT_VALUE = "${params.K8S_DEPLOYMENT}"
        K8S_CONTAINER_VALUE = "${params.K8S_CONTAINER}"
        K8S_NAMESPACE_VALUE = "${params.K8S_NAMESPACE}"
        K8S_CONTEXT_VALUE = "${params.K8S_CONTEXT}"

        /*
         * Used to determine whether an automatic Kubernetes
         * rollback is required after a deployment failure.
         */
        DEPLOYMENT_STARTED = 'false'
        DEPLOYMENT_COMPLETED = 'false'
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
                echo 'Running application validation and automated tests...'

                /*
                 * Jenkins is running inside Docker and uses the
                 * Docker socket to communicate with the Docker daemon.
                 *
                 * The Jenkins workspace is stored inside the
                 * jenkins_home Docker volume.
                 *
                 * The temporary Python container mounts that same
                 * volume so it can access the checked-out source code.
                 */
                sh '''
                    set -e

                    docker run --rm \
                        -v jenkins_home:/jenkins_home \
                        -w "/jenkins_home/workspace/finacplus-cicd" \
                        python:3.12-slim \
                        sh -c '
                            set -e

                            echo "Python version:"
                            python --version

                            echo "Installing application and test dependencies..."
                            pip install --no-cache-dir \
                                -r app/requirements.txt \
                                -r requirements-test.txt

                            echo "Running Python syntax validation..."
                            python -m py_compile app/app.py

                            echo "Running automated tests..."
                            PYTHONPATH=/jenkins_home/workspace/finacplus-cicd pytest tests/ -v
                        '
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image: ${params.IMAGE_REPO}:${BUILD_NUMBER}"

                sh '''
                    set -e

                    docker build \
                        -t "${IMAGE_REPO_VALUE}:${BUILD_NUMBER}" \
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

                        docker push "${IMAGE_REPO_VALUE}:${BUILD_NUMBER}"
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

                /*
                 * Mark deployment as started before modifying
                 * the Kubernetes Deployment.
                 */
                script {
                    env.DEPLOYMENT_STARTED = 'true'
                }

                sh '''
                    set -e

                    kubectl --context="${K8S_CONTEXT_VALUE}" \
                        --namespace="${K8S_NAMESPACE_VALUE}" \
                        set image deployment/"${K8S_DEPLOYMENT_VALUE}" \
                        "${K8S_CONTAINER_VALUE}"="${IMAGE_REPO_VALUE}:${BUILD_NUMBER}"

                    kubectl --context="${K8S_CONTEXT_VALUE}" \
                        --namespace="${K8S_NAMESPACE_VALUE}" \
                        rollout status deployment/"${K8S_DEPLOYMENT_VALUE}" \
                        --timeout=120s
                '''

                /*
                 * Mark the deployment complete only after
                 * Kubernetes reports a successful rollout.
                 */
                script {
                    env.DEPLOYMENT_COMPLETED = 'true'
                }
            }
        }

        stage('Deployment Verification') {
            steps {
                echo 'Verifying Kubernetes deployment...'

                sh '''
                    set -e

                    echo "Deployment status:"

                    kubectl --context="${K8S_CONTEXT_VALUE}" \
                        --namespace="${K8S_NAMESPACE_VALUE}" \
                        get deployment "${K8S_DEPLOYMENT_VALUE}"

                    echo "Pod status:"

                    kubectl --context="${K8S_CONTEXT_VALUE}" \
                        --namespace="${K8S_NAMESPACE_VALUE}" \
                        get pods \
                        -l app="${K8S_CONTAINER_VALUE}"
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
            script {

                /*
                 * Roll back only when Kubernetes deployment
                 * actually started but did not complete.
                 *
                 * Test, build, and push failures do not trigger
                 * a Kubernetes rollback.
                 */
                if (env.DEPLOYMENT_STARTED == 'true' &&
                    env.DEPLOYMENT_COMPLETED != 'true') {

                    echo 'Deployment failed. Attempting Kubernetes rollback...'

                    sh '''
                        set +e

                        kubectl --context="${K8S_CONTEXT_VALUE}" \
                            --namespace="${K8S_NAMESPACE_VALUE}" \
                            rollout undo deployment/"${K8S_DEPLOYMENT_VALUE}"

                        kubectl --context="${K8S_CONTEXT_VALUE}" \
                            --namespace="${K8S_NAMESPACE_VALUE}" \
                            rollout status deployment/"${K8S_DEPLOYMENT_VALUE}" \
                            --timeout=120s

                        echo "Rollback attempt completed."
                    '''
                }

                echo """
                Pipeline failed.

                Check the failed stage and its console output
                to identify the root cause.
                """
            }
        }

        always {
            echo "Build #${BUILD_NUMBER} completed."
        }
    }
}