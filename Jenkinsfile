pipeline {
    agent any

    tools {
        mvn 'Maven'
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t k8_app:latest .'
            }
        }
    }
}
