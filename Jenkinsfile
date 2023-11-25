pipeline{

  agent none

  environment {
        CI = 'false'
	GITHUB_REPO = '' //The github repo to pull the code from 
	GITHUB_CREDENTIALS = '' // github credentials if this is a private repo
	DOCKER_CREDENTIALS = '' // docker credentials to push the new docker image to
	KUBERNETES_URL = ''  // url of the api-server to ge
	KUBERNETES_NAMESPACE = 'default'
	KUBERNETES_CREDENTIALS = '' // kubernetes credentials to update
	DEPLOYMENT = '' // name of the deployment to update
	CONTAINTER_NAME = '' // name of the container in the deployment that needs an image update
    }

    stages {

        stage('Pull code from GitHub'){
            agent any
            steps {
                git url:"https://github.com/${env.GITHUB_REPO}", branch:'main',credentialsId:"${env.GITHUB_CREDENTIALS}"
            }
        }

        stage('Build'){
            agent {
                 docker {
                   image 'node:20.9.0-alpine3.18'
                 }
            }
            steps {
                sh 'npm install react-router-dom'
                sh 'npm install react-router-config'
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Write Dockerfile File to copy the built files to the nginx directory'){
            agent any
            steps {
                writeFile file: "Dockerfile" , text:"FROM nginx\nCOPY build/ /usr/share/nginx/html"
            }
        }

        stage('Build NGINX Docker'){
            agent any
            steps {
                sh "docker build -t nginx-frontend:$BUILD_NUMBER ."
            }
        }

        stage('Push to Docker Hub'){
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CREDENTIALS}", passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){               
                     script {			
                    	env.DOCKER_HUB_USER = "${env.dockerHubUser}"
		     }
                     sh "docker tag nginx-frontend:$BUILD_NUMBER ${env.dockerHubUser}/nginx-frontend:$BUILD_NUMBER"
                     sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                     sh "docker push ${env.dockerHubUser}/nginx-frontend:$BUILD_NUMBER"
               }
            }
        }

        stage('Update the deployment with the new image'){
                agent {
                   docker {
                     image 'portainer/kubectl-shell'
                    }
                 }
            steps {
                withKubeConfig([credentialsId: "${env.KUBERNETES_CREDENTIALS}",serverUrl: "${env.KUBERNETES_URL}" ,namespace: "${env.KUBERNETES_NAMESPACE}"]) { 
                     sh "kubectl set image deployment ${env.DEPLOYMENT} ${env.CONTAINTER_NAME}=${env.DOCKER_HUB_USER}/nginx-frontend:$BUILD_NUMBER"
               }
            }
        }

    }
}