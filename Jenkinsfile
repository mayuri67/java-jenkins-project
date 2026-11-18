pipeline {

    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {

        DOCKER_USER = 'mayuri67'

        IMAGE_NAME = 'java-app'

        IMAGE_TAG = "${BUILD_NUMBER}"

        DOCKER_CREDENTIALS = 'dockerhub-creds'

    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/mayuri67/java-jenkins-project.git'

            }
        }

        stage('Build') {

            steps {

                sh 'mvn clean compile'

            }
        }

        stage('Unit Test') {

            steps {

                sh 'mvn test'

            }
        }

        stage('Package') {

            steps {

                sh 'mvn package -DskipTests'

            }
        }

        stage('Docker Build') {

            steps {

                sh """
                docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_USER}/${IMAGE_NAME}:latest
                """

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
                    echo "$DOCKER_PASSWORD" | docker login \
                    -u "$DOCKER_USERNAME" --password-stdin

                    docker push ${DOCKER_USERNAME}/java-app:${BUILD_NUMBER}

                    docker push ${DOCKER_USERNAME}/java-app:latest

                    docker logout
                    '''

                }
            }
        }

        stage('Deploy to Kubernetes') {

            steps {

                withCredentials([
                    file(
                        credentialsId: 'microk8s-kubeconfig',
                        variable: 'KUBECONFIG_FILE'
                    )
                ]) {

                    sh '''
                    export KUBECONFIG=$KUBECONFIG_FILE

                    kubectl apply -f k8s/deployment.yaml

                    kubectl set image deployment/java-app \
                    java-app=mayuri67/java-app:${BUILD_NUMBER}

                    kubectl rollout status deployment/java-app \
                    --timeout=180s
                    '''

                }
            }
        }

    }

    post {

        success {

            echo 'Java CI/CD Pipeline Completed Successfully!'

        }

        failure {

            echo 'Java CI/CD Pipeline Failed! Check Jenkins Console Output.'

        }

        always {

            echo 'Pipeline Execution Completed.'

        }

    }

}
