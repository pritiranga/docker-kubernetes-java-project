pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t k8_app:latest -f shopfront/Dockerfile .'
            }
        }
    }
}
