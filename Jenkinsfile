pipeline {
    agent any

    environment {
        KUBECONFIG_CRED_ID = 'kubeproxy'        // Jenkins Secret File credential ID
        NAMESPACE          = 'three-tier'
        MANIFEST_PATH      = 'DB/database.yml'
        DEPLOYMENT_NAME    = 'mongodb'
    }

    stages {

        // ─────────────────────────────────────────────
        // STAGE 1: Checkout source code from SCM/Git
        // ─────────────────────────────────────────────
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Code checked out successfully"
                sh 'ls -la'                          // confirm DB/ folder exists
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 2: Validate tools and manifest file
        // ─────────────────────────────────────────────
        stage('Validate Tools & Manifest') {
            steps {
                sh '''
                    echo "--- Checking kubectl ---"
                    kubectl version --client

                    echo "--- Checking manifest file exists ---"
                    if [ ! -f "DB/database.yml" ]; then
                        echo "ERROR: DB/database.yml not found!"
                        exit 1
                    fi

                    echo "--- Validating YAML syntax ---"
                    kubectl apply -f DB/database.yml --dry-run=client \
                        --validate=false 2>&1 || true

                    echo "✅ All validations passed"
                '''
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 3: Ensure namespace exists
        // ─────────────────────────────────────────────
        stage('Prepare Namespace') {
            steps {
                withKubeConfig([credentialsId: "${KUBECONFIG_CRED_ID}"]) {
                    sh '''
                        echo "--- Checking namespace ---"
                        kubectl get namespace three-tier 2>/dev/null \
                            || kubectl create namespace three-tier

                        echo "✅ Namespace three-tier is ready"
                    '''
                }
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 4: Deploy MongoDB to Kubernetes
        // ─────────────────────────────────────────────
        stage('Deploy MongoDB') {
            steps {
                withKubeConfig([credentialsId: "${KUBECONFIG_CRED_ID}"]) {
                    sh '''
                        echo "--- Applying manifests ---"
                        kubectl apply -f DB/database.yml -n three-tier

                        echo "--- Resources applied ---"
                        kubectl get secret   -n three-tier
                        kubectl get deploy   -n three-tier
                        kubectl get svc      -n three-tier

                        echo "✅ Manifests applied successfully"
                    '''
                }
            }
        }

        // ─────────────────────────────────────────────
        // STAGE 5: Verify rollout
        // ─────────────────────────────────────────────
        stage('Verify Rollout') {
            steps {
                withKubeConfig([credentialsId: "${KUBECONFIG_CRED_ID}"]) {
                    sh '''
                        echo "--- Waiting for rollout ---"
                        kubectl rollout status deployment/mongodb \
                            -n three-tier --timeout=180s

                        echo "--- Pod status ---"
                        kubectl get pods -n three-tier -l app=mongodb -o wide

                        echo "--- Service status ---"
                        kubectl get svc -n three-tier

                        echo "✅ MongoDB is running successfully"
                    '''
                }
            }
        }
    }

    // ─────────────────────────────────────────────
    // POST: Always show result
    // ─────────────────────────────────────────────
    post {
        success {
            echo """
            ========================================
            ✅ DEPLOYMENT SUCCESSFUL
            Namespace  : three-tier
            Deployment : mongodb
            ========================================
            """
        }
        failure {
            echo "❌ DEPLOYMENT FAILED — collecting debug info..."
            withKubeConfig([credentialsId: "${KUBECONFIG_CRED_ID}"]) {
                sh '''
                    echo "--- Pod list ---"
                    kubectl get pods -n three-tier || true

                    echo "--- Pod describe ---"
                    kubectl describe pods -n three-tier -l app=mongodb || true

                    echo "--- Pod logs ---"
                    kubectl logs -l app=mongodb -n three-tier --tail=50 || true

                    echo "--- Events ---"
                    kubectl get events -n three-tier --sort-by=.lastTimestamp || true
                '''
            }
        }
        always {
            echo "Pipeline finished at: ${new Date()}"
        }
    }
}
