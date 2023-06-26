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

	stage('Creating namespace on k8 cluster') {
		steps {
  	              sshagent(['k8-server']){
                      	sh 'ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 "kubectl create ns k8-task"'
		      }
		}
	}

	stage('Deploy application') {
		steps {
			sshagent(['k8-server']){
				kubernetesDeploy(
                        		configs: 'k8-task.yml',
                        		kubeconfigId: 'kubeconfig',
                        		enableConfigSubstitution: true 
                    		)
			}
		}
	}
				

    }
}
