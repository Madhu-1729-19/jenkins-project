pipeline {
    agent any

    environment {
        KUBE_NAMESPACE = "three-tier"
        KUBE_FILE = "DB/database.yml"
        KUBECONFIG = "/var/jenkins_home/.kube/config"
    }

    stages {

        stage('Verify Kubernetes Access') {
            steps {
                echo 'Checking kubectl connection...'
                sh '''
                kubectl get nodes
                '''
            }
        }

        stage('Create Namespace (if not exists)') {
            steps {
                echo 'Creating namespace if not present...'
                sh '''
                kubectl create namespace $KUBE_NAMESPACE || true
                '''
            }
        }

        stage('Deploy MongoDB') {
            steps {
                echo 'Deploying MongoDB to Kubernetes Namespace three-tier'

                sh '''
                kubectl apply -f $KUBE_FILE -n $KUBE_NAMESPACE
                '''
            }
        }

    }

    post {
        success {
            echo 'Deployment Successful ✔'
        }
        failure {
            echo 'Deployment Failed ❌ - check kubeconfig or YAML'
        }
    }
}
