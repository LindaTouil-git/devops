pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        IMAGE_NAME = "lindatouil/lindatouil_devops"
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS = 'docker-hub'
        SONAR_TOKEN = credentials('sonar-token')

    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Récupération du dépôt Git..."
                checkout scm
            }
        }

        stage('Clean Workspace') {
            steps {
                echo "Nettoyage du workspace..."
                sh 'git clean -fdx'
            }
        }

        stage('Build Project') {
            steps {
                echo "Build Maven du projet..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Analyse qualité du code avec SonarQube..."
                withSonarQubeEnv('SonarQube Server') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=lindatouil_devops \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Construction de l'image Docker..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Push de l'image sur Docker Hub..."
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS,
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin ${REGISTRY}
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout ${REGISTRY}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Déploiement sur Kubernetes (namespace devops)..."
                sh 'kubectl apply -f k8s/mysql-deployment.yaml -n devops'
                sh 'kubectl apply -f k8s/spring-deployment.yaml -n devops'
                sh 'kubectl rollout restart deployment spring-app -n devops'
                sh 'kubectl get pods -n devops'
                sh 'kubectl get services -n devops'
            }
        }
    }

    post {
        success {
            echo "Pipeline terminé avec succès."
        }
        failure {
            echo "Échec du pipeline. Vérifie les logs."
        }
    }
}
