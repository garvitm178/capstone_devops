pipeline {
	agent any
	stages {
		stage('Create kubernetes cluster') {
			steps {
				withAWS(region:'us-east-1', credentials:'credentials') {
					sh '''
						eksctl create cluster \
						--name capstonecluster \
						--version 1.13 \
						--nodegroup-name standard-workers \
						--node-type t2.micro \
						--nodes 2 \
						--nodes-min 1 \
						--nodes-max 3 \
						--node-ami auto \
						--region us-east-1 \
						--zones us-east-1a \
						--zones us-east-1b \
						--zones us-east-1c \
					'''
				}
			}
		}

		

		stage('Create config file cluster') {
			steps {
				withAWS(region:'us-east-1', credentials:'credentials') {
					sh '''
						aws eks --region us-east-1 update-kubeconfig --name capstonecluster
					'''
				}
			}
		}

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker') {
			steps {
				script {
          			app_image = docker.build("garvitmehta/capstone")
        		}
			}
		}

		stage('Push Image') {
			steps {
				script {
          			docker.withRegistry('https://registry.hub.docker.com', 'garvitmehta') {
            			app_image.push("${env.GIT_COMMIT[0..7]}")
            			app_image.push("latest")
          			}
        		}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-east-1', credentials:'credentials') {
					sh '''
						kubectl config use-context arn:aws:eks:us-east-1:142977788479:cluster/capstonecluster
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-1', credentials:'credentials') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-1', credentials:'credentials') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-1', credentials:'credentials') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Create the service, redirect to green') {
			steps {
				withAWS(region:'us-east-1', credentials:'credentials') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}
	}
}
