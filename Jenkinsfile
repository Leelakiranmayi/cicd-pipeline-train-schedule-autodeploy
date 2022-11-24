/* Hello World in Groovy */
pipeline {
    agent none
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gupta1299/train-schedule"  
		DOCKERHUB_CREDENTIALS=credentials('guota1299')
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

                sh 'sudo docker build -t gupta1299/train-schedule .'

            }
        }
        stage('Push Docker Image') {
            agent {
                label 'java-slave'
            }
            steps {
		sh 'sudo chmod 666 /var/run/docker.sock'
                sh 'sudo echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'sudo docker push gupta1299/train-schedule'
            }
        }
        stage('CanaryDeploy') {
            agent {
                label 'kubernetes'
            }
            environment {
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            agent {
                label 'kubernetes'
            }
            environment {
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
