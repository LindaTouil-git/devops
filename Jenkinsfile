pipeline {
    agent any
    environment {
        REGISTRY = "docker.io"          // Ton registre (ex: Docker Hub)
        IMAGE_NAME = "lindatouil/lindatouil_devops" // Nom de ton image
        IMAGE_TAG = "latest"            // Tag de l'image
        DOCKER_CREDENTIALS = 'docker-hub' // ID des credentials Jenkins
    }
    triggers {
        // Le webhook GitHub déclenche déjà, mais on met un fallback
        pollSCM('* * * * *') // Vérifie chaque minute au cas où
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
                echo "Reconstruction du projet..."
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Construction de l'image Docker..."
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }
        stage('Push Docker Image') {
            steps {
                echo "Connexion & push sur le registre..."
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS,
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
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
                echo "Déploiement sur Kubernetes..."
                sh 'kubectl apply -f mysql-deployment.yaml'
                sh 'kubectl apply -f spring-deployment.yaml'
                sh 'kubectl rollout restart deployment/spring-app -n devops'
                echo "Vérification du déploiement..."
                sh 'kubectl get pods -n devops'
                sh 'kubectl get svc -n devops'
            }
        }
    }
    post {
        success {
            echo "Pipeline terminé avec succès 🎉"
        }
        failure {
            echo "Le pipeline a échoué ❌"
        }
    }
}
