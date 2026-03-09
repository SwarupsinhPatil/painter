pipeline {
    agent { label 'k3s' }

    environment {
        DOCKER_IMAGE = "swarupsinh10/painter:latest"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/SwarupsinhPatil/painter.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl delete deployment painter --ignore-not-found
                kubectl delete service painter --ignore-not-found
                kubectl create deployment painter --image=$DOCKER_IMAGE
                kubectl expose deployment painter --type=NodePort --port=80
                '''
            }
        }

        stage('Check Deployment') {
            steps {
                sh 'kubectl get pods'
                sh 'kubectl get svc'
            }
        }

    }
}
