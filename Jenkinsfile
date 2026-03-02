pipeline {
    agent any

    environment {
        IMAGE_NAME = "swarupsinh10/painter"
        DOCKER_CRED = "dockerhub-cred"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/SwarupsinhPatil/painter.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CRED, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $IMAGE_NAME:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                  export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
                  kubectl create deployment painter --image=swarupsinh10/painter:latest --replicas=3 --dry-run=client -o yaml | kubectl apply -f -
                  kubectl expose deployment painter --type=NodePort --port=80 --dry-run=client -o yaml | kubectl apply -f -
                  '''
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker system prune -f'
            }
        }
    }
}
