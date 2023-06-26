pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Build') {
            steps {
                sh 'cd shopfront'
                sh 'docker build -t k8_app:latest .'
            }
        }
    }
}
