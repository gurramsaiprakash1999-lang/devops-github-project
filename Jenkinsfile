pipeline {
    agent any
    stages {
        stage('1. Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/gurramsaiprakash1999-lang/devops-github-project.git'
            }
        }
        stage('2. Build Docker Image') {
            steps {
                sh 'docker build --no-cache -t webapp:latest .'
            }
        }
        stage('3. Deploy to Kubernetes Cluster') {
            steps {
                sh '''
                    export PATH=$PATH:/usr/local/bin
                    export KUBECONFIG=/var/lib/jenkins/.kube/config
                    
                    # Ensure Minikube image registry receives the new build
                    minikube image load webapp:latest
                    
                    # Apply Manifests & Force Pod Refresh
                    kubectl apply -f k8s-deployment.yaml
                    kubectl rollout restart deployment/webapp-deployment
                    kubectl rollout status deployment/webapp-deployment --timeout=60s
                '''
            }
        }
    }
    post {
        always {
            withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    COMMIT_HASH=$(git rev-parse HEAD)
                    
                    curl -s -X POST \
                      -H "Authorization: Bearer $GITHUB_TOKEN" \
                      -H "Accept: application/vnd.github.v3+json" \
                      https://api.github.com/repos/gurramsaiprakash1999-lang/devops-github-project/statuses/$COMMIT_HASH \
                      -d "{\\"state\\": \\"success\\", \\"target_url\\": \\"${BUILD_URL}\\", \\"description\\": \\"Deployed to Kubernetes NodePort 30300\\", \\"context\\": \\"continuous-integration/jenkins\\"}"
                '''
            }
        }
    }
}
