
pipeline {

    agent any

    environment {

        DOCKER_USER = 'sam3583558'

        DOCKER_CREDENTIAL = 'dockerhub-creds'

        WEB_IMAGE = 'sam3583558/ecom-web'
        DB_IMAGE  = 'sam3583558/ecom-db'

        K8S_NAMESPACE = 'ecom'

        WEB_YAML = 'k8s_yaml\\ecom-web-deployment.yaml'
        DB_YAML  = 'k8s_yaml\\ecom-db-statefulset.yaml'
        SERVICE_YAML = 'k8s_yaml\\ecom-web-service.yaml'
    }

    stages {

        stage('Checkout Developer Code') {
            steps {

                git branch: 'main',
                    url: 'https://github.com/sam358-crypto/ecom-app3tier.git'

            }
        }

        stage('Show Version') {
            steps {

                echo "=========================================="
                echo "JENKINS BUILD NUMBER: ${BUILD_NUMBER}"
                echo "WEB IMAGE: ${WEB_IMAGE}:${BUILD_NUMBER}"
                echo "DB IMAGE : ${DB_IMAGE}:${BUILD_NUMBER}"
                echo "=========================================="
            }
        }

        stage('Build Web Image') {
            steps {

                bat """
                    docker build ^
                    -f Dockerfile.web ^
                    -t ${WEB_IMAGE}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Build DB Image') {
            steps {

                bat """
                    docker build ^
                    -f Dockerfile.db ^
                    -t ${DB_IMAGE}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Docker Login') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIAL}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    bat '''
                        echo %DOCKER_PASSWORD% | docker login ^
                        -u %DOCKER_USERNAME% ^
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {

                bat """
                    docker push ${WEB_IMAGE}:${BUILD_NUMBER}

                    docker push ${DB_IMAGE}:${BUILD_NUMBER}
                """
            }
        }

        stage('Prepare Kubernetes YAML') {
            steps {

                bat """
                    powershell -NoProfile -Command ^
                    "(Get-Content '%WEB_YAML%') -replace 'VERSION','%BUILD_NUMBER%' | Set-Content '%WORKSPACE%\\k8s_yaml\\ecom-web-deployment-version.yaml'"

                    powershell -NoProfile -Command ^
                    "(Get-Content '%DB_YAML%') -replace 'VERSION','%BUILD_NUMBER%' | Set-Content '%WORKSPACE%\\k8s_yaml\\ecom-db-statefulset-version.yaml'"
                """
            }
        }

        stage('Deploy DB') {
            steps {

                bat """
                    kubectl apply ^
                    -f "%WORKSPACE%\\k8s_yaml\\ecom-db-statefulset-version.yaml" ^
                    -n ${K8S_NAMESPACE}
                """
            }
        }

        stage('Deploy Web') {
            steps {

                bat """
                    kubectl apply ^
                    -f "%WORKSPACE%\\k8s_yaml\\ecom-web-deployment-version.yaml" ^
                    -n ${K8S_NAMESPACE}

                    kubectl apply ^
                    -f "%WORKSPACE%\\k8s_yaml\\ecom-web-service.yaml" ^
                    -n ${K8S_NAMESPACE}
                """
            }
        }

        stage('Wait for Deployment') {
            steps {

                bat """
                    kubectl rollout status ^
                    deployment/ecom-web ^
                    -n ${K8S_NAMESPACE} ^
                    --timeout=5m
                """
            }
        }

        stage('Verify') {
            steps {

                bat """
                    echo ==========================================
                    echo PODS
                    echo ==========================================

                    kubectl get pods -n ${K8S_NAMESPACE} -o wide

                    echo ==========================================
                    echo SERVICES
                    echo ==========================================

                    kubectl get svc -n ${K8S_NAMESPACE}

                    echo ==========================================
                    echo WEB DEPLOYMENT
                    echo ==========================================

                    kubectl get deployment ecom-web -n ${K8S_NAMESPACE}

                    echo ==========================================
                    echo DB STATEFULSET
                    echo ==========================================

                    kubectl get statefulset ecom-db -n ${K8S_NAMESPACE}

                    echo ==========================================
                    echo DEPLOYED VERSION
                    echo ==========================================

                    kubectl get deployment ecom-web ^
                    -n ${K8S_NAMESPACE} ^
                    -o jsonpath="{.spec.template.spec.containers[*].image}"

                    echo.
                """
            }
        }
    }

    post {

        success {
            echo """
            ==========================================
            DEPLOYMENT SUCCESSFUL
            ==========================================

            Build Number : ${BUILD_NUMBER}

            Web Image    : ${WEB_IMAGE}:${BUILD_NUMBER}

            DB Image     : ${DB_IMAGE}:${BUILD_NUMBER}

            Kubernetes   : ${K8S_NAMESPACE}

            ==========================================
            """
        }

        failure {
            echo """
            ==========================================
            DEPLOYMENT FAILED
            ==========================================

            Build Number : ${BUILD_NUMBER}

            Check Jenkins Console Output.

            ==========================================
            """
        }

        always {
            bat 'docker logout || exit 0'

            bat """
                if exist "%WORKSPACE%\\k8s_yaml\\ecom-web-deployment-version.yaml" del "%WORKSPACE%\\k8s_yaml\\ecom-web-deployment-version.yaml"

                if exist "%WORKSPACE%\\k8s_yaml\\ecom-db-statefulset-version.yaml" del "%WORKSPACE%\\k8s_yaml\\ecom-db-statefulset-version.yaml"
            """
        }
    }
}
