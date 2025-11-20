pipeline {
    agent any

    stages {

        stage('MAVEN') {
            steps {
                sh "mvn -version"
            }
        }

        stage('GIT') {
            steps {
                git branch: 'main', url: 'https://github.com/ahmed0199/DevOps.git'
            }
        }

    }
}