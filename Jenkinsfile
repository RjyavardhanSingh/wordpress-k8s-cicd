pipeline {
    agent any

    environment {
        KUBECONFIG = "/var/jenkins_home/.kube/config"
        NAMESPACE = "wordpress-cicd"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/RjyavardhanSingh/wordpress-k8s-cicd.git'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment/namespace.yaml
                kubectl apply -f deployment/mysql-deployment.yaml
                kubectl apply -f deployment/wordpress-deployment.yaml
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'kubectl get pods -n ${NAMESPACE}'
            }
        }
    }
}
