pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKERHUB_USERNAME = 'etinosaizekor'
        IMAGE_NAME = 'cw2-server'
        IMAGE_TAG = '1.0'
    }
    
    stages {
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Image') {
            steps {
                sh 'docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }
        
        stage('Test Image') {
            steps {
                sh '''
                    docker run -d --name test-container -p 8081:8081 ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 5
                    docker exec test-container curl -s http://localhost:8081
                    docker stop test-container
                    docker rm test-container
                '''
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.166 \
                    "kubectl set image deployment/cw2-server cw2-server=etinosaizekor/cw2-server:${BUILD_NUMBER} && \
                    kubectl rollout status deployment/cw2-server"
                """
            }
        }
    }
    
    post {
        always {
            sh 'docker logout'
        }
    }
}
