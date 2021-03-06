pipeline {
  agent any
  stages {
    stage('Lint HTML') {
      steps {
        sh 'tidy -q -e *.html'
      }
    }

    stage('Build Docker Image') {
      steps {
        withCredentials(bindings: [[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]) {
          sh '''
						docker build -t mjpraino/capstone-project .
					'''
        }

      }
    }

    stage('Push Image To Dockerhub') {
      steps {
        withCredentials(bindings: [[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]) {
          sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push mjpraino/capstone-project
					'''
        }

      }
    }

    stage('Kube config') {
      steps {
        withAWS(region: 'us-west-2', credentials: 'Capstone-EKS') {
          sh '''
						aws2 eks --region us-west-2 update-kubeconfig --name capstonecluster
						kubectl config use-context arn:aws:eks:us-west-2:292152339671:cluster/capstonecluster'''
        }

      }
    }

    stage('Deploy blue container') {
      steps {
        withAWS(region: 'us-west-2', credentials: 'Capstone-EKS') {
          sh '''
	                                        aws2 sts get-caller-identity
						kubectl apply -f ./blue-controller.json
					'''
        }

      }
    }

    stage('Deploy green container') {
      steps {
        withAWS(region: 'us-west-2', credentials: 'Capstone-EKS') {
          sh '''
						kubectl apply -f ./green-controller.json
					'''
        }

      }
    }

    stage('Create the service in the cluster, redirect to blue') {
      steps {
        withAWS(region: 'us-west-2', credentials: 'Capstone-EKS') {
          sh '''
						kubectl apply -f ./blue-service.json
					'''
        }

      }
    }

    stage('Pending User Approval') {
      steps {
        input 'Ready to redirect traffic to green?'
      }
    }

    stage('Create the service in the cluster, redirect to green') {
      steps {
        withAWS(region: 'us-west-2', credentials: 'Capstone-EKS') {
          sh '''
						kubectl apply -f ./green-service.json
					'''
        }

      }
    }

  }
}