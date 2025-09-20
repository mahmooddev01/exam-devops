pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        IMAGE_NAME = 'mahmooddev/examen_devops'
        IMAGE_TAG = 'latest'
        RENDER_API_KEY = credentials('render-api-key')
        RENDER_SERVICE_ID = 'srv-d37i7rmr433s73epn4jg'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mahmooddev01/exam-devops'
            }
        }

        stage('Test Maven') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build & Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}", "--pull .")
                }
            }
        }

        stage('Push Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Render') {
            steps {
                echo "Déploiement sur Render via Docker..."
                script {
                    retry(3) {
                        sh '''
                        curl -X POST \
                          -H "Accept: application/json" \
                          -H "Authorization: Bearer ${RENDER_API_KEY}" \
                          -d '{ "dockerImage": "${IMAGE_NAME}:${IMAGE_TAG}" }' \
                          https://api.render.com/v1/services/${RENDER_SERVICE_ID}/deploys
                        '''
                    }
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Build, test ou déploiement échoué'
        }
        success {
            echo '✅ Build, push Docker et déploiement Render terminés'
        }
    }
}
