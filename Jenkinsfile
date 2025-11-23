pipeline {
    agent any

    stages {

        stage('MAVEN') {
            steps {
                sh "mvn -version"
            }
        } // message

        stage('GIT') {
            steps {
                git branch: 'main', url: 'https://github.com/LindaTouil-git/devops.git'
            }
        }

    }
}
