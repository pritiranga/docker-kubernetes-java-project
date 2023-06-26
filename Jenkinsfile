pipeline {
	agent any

	tools {
    		maven 'Maven'
    	}

	environment {
		DOCKERHUB=credentials('Dockerhub')
    	}
    
	parameters {
    		booleanParam(name: 'enableCleanUp', defaultValue: false, description: 'Select to clean the environements')
    	}

	stages {

		stage('Check if Environment exists') {
			steps{
				script{
               				try {
                        			sshagent(['k8-server']) {
                            				def result = sh(
                                				script: 'ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 "kubectl get namespace k8-task"',
                                				returnStatus: true
                            				)
                            				if (result == 0) {
                                				echo 'Environment exists. Skipping to cleanup approval stage.'
                                				buildCleanupApprovalStage()
                            				} else {
                                				echo 'Environment does not exist. Proceeding with all stages.'
                                				buildAllStages()
                            				}
                        			}
                    			} catch (Exception e) {
                        			echo 'Error occurred while checking the environment.'
                        			currentBuild.result = 'FAILURE'
                    			}
                		}
            		}
        	}
    }
        
		stage('Build') {
	    		when {
                    		expression{
                        		params.enableCleanUp == false
                    		}
                	}
            		steps {
                		sh 'docker build -t k8_app:latest -f shopfront/Dockerfile .'
            		}
        	}

		stage('Publish to Dockerhub') {
			when {
                    		expression{
                        		params.enableCleanUp == false
                    		}
                	}
            		steps{
                		sh 'docker tag k8_app:latest pritidevops/k8_app:latest'
                		sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                		sh 'docker push pritidevops/k8_app:latest'
            		}
        	}

		stage('Creating namespace on k8 cluster') {
                	when {
                    		expression{
                    		    	params.enableCleanUp == false
                    		}
                	}
			steps {
  	              		sshagent(['k8-server']){
                      			sh 'ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 "kubectl create ns k8-task"'
		      		}
			}
		}

		stage('Deploy application') {
                	when {
                   		expression{
                        		params.enableCleanUp == false
                    		}
                	}
			steps {
				sshagent(['k8-server']){
					kubernetesDeploy(
                        			configs: 'shopfront/k8-task.yml',
                        			kubeconfigId: 'kubeconfig',
                        			enableConfigSubstitution: true 
                    			)
				}
			}
		}

		stage('Clean Up Approval'){
                	steps{              
                	    	script {
                        		timeout(time: 10, unit: 'MINUTES'){
                            			input ('Proceed with Environment CleanUp?')
                        		}
                    		}        
                	} 
            	}

		stage('Cleanup'){
                	when {
                    		expression{
                        		params.enableCleanUp == true
                    		}
                	}
			steps {
  	              		sshagent(['k8-server']){
                      			sh 'ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 "kubectl delete ns k8-task"'
		      		}
			}
		}

    } //stages
} //pipeline
