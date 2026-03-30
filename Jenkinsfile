pipeline {
agent any


environment {
    DOCKER_IMAGE = "ganeswara/flask-devsecops"
    TAG = "latest"
}

    stages {

            stage('SAST - Bandit') {
                steps {
                sh '''
                bandit -r app/ -ll
                '''
        }
    }

    stage('Build Docker Image') {
        steps {
                sh '''
                export DOCKER_BUILDKIT=0
                docker build -t ganeswara/flask-devsecops:latest .
                '''
        }
}

    stage('SCA - Trivy Scan') {
        steps {
            sh '''
            trivy --download-db-only
            trivy image ganeswara/flask-devsecops:latest
            '''
    }
}


    stage('Docker Login') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                sh 'echo $PASS | docker login -u $USER --password-stdin'
            }
        }
    }

    stage('Push Image') {
        steps {
            sh 'docker push $DOCKER_IMAGE:$TAG'
        }
    }

    stage('Deploy to Kubernetes') {
        steps {
            sh 'kubectl apply -f k8s/'
            sh 'kubectl rollout restart deployment flask-devsecops'
        }
    }

    stage('DAST - OWASP ZAP') {
        steps {
            sh 'docker run -t zaproxy/zap-stable zap-baseline.py -t http://host.docker.internal:30007'
        }
    }
}


}
 