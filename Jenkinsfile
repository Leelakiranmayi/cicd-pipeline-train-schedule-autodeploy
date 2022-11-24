/* Hello World in Groovy */
pipeline {
    agent none
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gupta1299/train-schedule"  
        DOCKERHUB_CREDENTIALS=credentials('docker')
    }
    stages {
        stage('Build') {
            agent {
                label 'java-slave'
            }
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            agent {
                label 'java-slave'
            }
            steps {

                sh 'sudo docker build -t train-schedule .'
		sh 'docker image list'
		sh 'docker tag train-schedule gupta1299/train-schedule'

            }
        }
        stage('Push Docker Image') {
            agent {
                label 'java-slave'
            }
            steps {
		sh 'sudo chmod 666 /var/run/docker.sock'
                sh 'sudo docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW docker.io'
                sh 'sudo docker push gupta1299/train-schedule:latest'
            }
        }
	stage('Deployement') {
            agent {
                label 'kubernetes'
            }
            steps {
                sh 'sudo kubectl apply -f train-schedule-kube-canary.yml'
	    }
        }   
    }
}
