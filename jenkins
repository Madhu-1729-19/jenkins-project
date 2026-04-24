pipeline {
    agent any

    environment {
        KUBE_NAMESPACE = "three-tier"
        KUBE_FILE = "DB/database.yml"
    }

    stages {
        stage('Deploy MongoDB') {
            steps {
                echo 'Deploying MongoDB to Kubernetes Namespace three-tier'

                // Apply Kubernetes YAML
                sh '''
                kubectl apply -f $KUBE_FILE -n $KUBE_NAMESPACE
                '''
            }
        }
    }
}
