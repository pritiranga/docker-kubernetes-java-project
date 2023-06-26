pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
		DOCKERHUB=credentials('Dockerhub')
	}

    stages {
        
        stage('Build') {
            steps {
                sh 'docker build -t k8_app:latest -f shopfront/Dockerfile .'
            }
        }

        stage('Publish to Dockerhub') {
            steps{
                sh 'docker tag k8_app:latest pritidevops/k8_app:latest'
                sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                sh 'docker push pritidevops/k8_app:latest'
            }
        }
       
    }
}
