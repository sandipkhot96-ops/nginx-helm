pipeline {
    agent {
        kubernetes {
            label 'helm-nginx'
            defaultContainer 'helm'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: helm
    image: sandipkhot96/jenkins-agent-k8s:latest
    command:
    - cat
    tty: true
  # Uncomment if your Docker Hub image is private
  # imagePullSecrets:
  #   - name: regcred
"""
        }
    }

    environment {
        NAMESPACE = "nginx"
        RELEASE   = "nginx"
        CHART     = "bitnami/nginx"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Helm version check') {
            steps {
                container('helm') {
                    sh 'helm version'
                }
            }
        }

        stage('Add Helm repo') {
            steps {
                container('helm') {
                    sh '''
                      helm repo add bitnami https://charts.bitnami.com/bitnami
                      helm repo update
                    '''
                }
            }
        }

        stage('Helm dry-run (NO changes)') {
            steps {
                container('helm') {
                    sh '''
                      helm upgrade --install $RELEASE $CHART \
                        --namespace $NAMESPACE \
                        --create-namespace \
                        --dry-run
                    '''
                }
            }
        }

        stage('Deploy NGINX (Helm only)') {
            steps {
                container('helm') {
                    sh '''
                      helm upgrade --install $RELEASE $CHART \
                        --namespace $NAMESPACE \
                        --create-namespace \
                        --wait \
                        --timeout 5m
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ NGINX deployed successfully using Bitnami Helm chart"
        }
        failure {
            echo "❌ Deployment failed — no destructive action taken"
        }
    }
}
