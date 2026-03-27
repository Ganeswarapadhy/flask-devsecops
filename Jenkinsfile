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
            pip install bandit
            bandit -r . -ll
            '''
        }
    }

    stage('Build Docker Image') {
        steps {
            sh 'docker build -t $DOCKER_IMAGE:$TAG .'
        }
    }

    stage('SCA - Trivy Scan') {
        steps {
            sh '''
            docker run --rm aquasec/trivy image \
            --severity HIGH,CRITICAL \
            $DOCKER_IMAGE:$TAG
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
            sh '''
            kubectl apply -f k8s/
            kubectl rollout restart deployment flask-devsecops
            '''
        }
    }

    stage('DAST - OWASP ZAP') {
        steps {
            sh '''
            docker run -t owasp/zap2docker-stable zap-baseline.py \
            -t http://host.docker.internal:30007
            '''
        }
    }
}
```

}
