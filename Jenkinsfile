pipeline {
    agent any

    environment {
        IMAGE_NAME = "java-app"
        IMAGE_TAG = "1.0"
        REGISTRY = "docker.io/ateeq10"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mdatiq321/prac60.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -Dmaven.repo.local=.m2'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Image to Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to Docker Swarm') {
            steps {
                sh '''
                    docker service rm java-service || true

                    docker service create \
                        --name java-service \
                        --replicas 2 \
                        --network app-network \
                        $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
