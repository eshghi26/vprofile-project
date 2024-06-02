def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    environment {
	    DOCKER_HUB_CREDENTIALS = 'dockerhubcred' // ID of the Docker Hub credentials in Jenkins
        DOCKER_HUB_REPO = 'eshghi26/vprofile'
        DEPLOY_HOST = '192.168.56.17'
        DEPLOY_USER = 'vagrant'
	}
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
          }  
        }

        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // Build a Docker image for the Flask application
                    // def app = docker.build("${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}", "./Docker-files/app/multistage/")
                    dockerImage = docker.build( DOCKER_HUB_REPO + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS) {
                        // Push the Docker image to Docker Hub
                        // def app = docker.image("${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}")
                        // app.push()
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy To Host') {
            steps {
                script {
                    sshagent (credentials: ['dockervmpk']) {
                        sh """
                        ssh ${DEPLOY_USER}@${DEPLOY_HOST} << 'EOF'
                        docker login -u 'eshghi26' -p 'Amirhossein@26'
                        docker pull ${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}
                        docker stop vprofile || true
                        docker rm vprofile || true
                        docker run -d --name vprofile -p 80:8080 ${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}
EOF
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#java-project',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }


    }
}
