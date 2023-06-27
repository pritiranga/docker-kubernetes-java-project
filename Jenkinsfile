pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        DOCKERHUB = credentials('Dockerhub')
    }

    parameters {
        booleanParam(
            name: 'enableCleanUp',
            defaultValue: false,
            description: 'Select to clean the environments'
        )
    }

    stages {
        stage('Check if Environment exists') {
            steps {
                script {
                    try {
                        sshagent(['k8-server']) {
                            def result = sh(
                                script: 'ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 "kubectl get namespace k8-task"',
                                returnStatus: true
                            )
                            if (result == 0) {
                                echo 'Environment exists. Skipping to cleanup approval stage.'
                                return
                            }
                        }
                    } catch (Exception e) {
                        echo 'Error checking if environment exists: ' + e.getMessage()
                        currentBuild.result = 'FAILURE'
                        error "Error checking if environment exists: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Cleanup Approval') {
            when {
                expression { params.enableCleanUp }
            }
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        input message: 'Do you want to cleanup the environment?', ok: 'Yes', rejectValue: 'No'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'DOCKERHUB', variable: 'DOCKERHUB_CRED')]) {
                        sh '''
                            docker build -t my-image:${env.BUILD_NUMBER} .
                            docker login -u my-dockerhub-user -p $DOCKERHUB_CRED
                            docker push my-image:${env.BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sshagent(['k8-server']) {
                        sh '''
                            kubectl apply -f deployment.yaml
                            kubectl rollout status deployment/my-app
                        '''
                    }
                }
            }
        }
    }
}
