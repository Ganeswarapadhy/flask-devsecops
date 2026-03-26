pipeline {
    agent any

    stages {

        stage('Clone Repo') {
            steps {
                git 'https://github.com/<your-username>/flask-devsecops.git'
            }
        }

        stage('SAST - Bandit') {
            steps {
                sh 'pip install bandit'
                sh 'bandit -r .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-devsecops .'
            }
        }

        stage('SCA - Trivy Image Scan') {
            steps {
                sh 'docker run --rm aquasec/trivy image flask-devsecops'
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker tag flask-devsecops <dockerhub-username>/flask-devsecops
                docker push <dockerhub-username>/flask-devsecops
                '''
            }
        }

        stage('DAST - OWASP ZAP') {
            steps {
                sh '''
                docker run -t owasp/zap2docker-stable zap-baseline.py \
                -t http://localhost:5000
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}