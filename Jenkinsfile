pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'flask-app'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building application...'
                sh '''
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh '''
                    source venv/bin/activate
                    # Add your test commands here
                    # pytest tests/
                '''
            }
        }
        
        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }
        
        stage('Docker Test') {
            steps {
                echo 'Testing Docker container...'
                sh """
                    docker run -d -p 5000:5000 --name flask-test ${DOCKER_IMAGE}:${DOCKER_TAG}
                    sleep 5
                    curl -f http://localhost:5000/health || exit 1
                    docker stop flask-test
                    docker rm flask-test
                """
            }
        }
        
        stage('Deploy to Kubernetes') {
            when {
                branch 'main' || branch 'master'
            }
            steps {
                echo 'Deploying to Kubernetes...'
                sh """
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    kubectl set image deployment/flask-app flask-app=${DOCKER_IMAGE}:${DOCKER_TAG}
                    kubectl rollout status deployment/flask-app
                """
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            sh 'docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

