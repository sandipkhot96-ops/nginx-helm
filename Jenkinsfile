pipeline {
    agent {
        kubernetes {
            label 'helm-nginx'
            defaultContainer 'helm'
            yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins-sa
  containers:
    - name: helm
      image: sandipkhot96/jenkins-agent-k8s:latest
      command:
        - sleep
        - infinity
      tty: true
"""
        }
    }

    environment {
        NAMESPACE = "nginx"
        RELEASE   = "nginx"
        CHART     = "bitnami/nginx"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Helm version') {
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
                      helm repo add bitnami https://charts.bitnami.com/bitnami || true
                      helm repo update
                    '''
                }
            }
        }

        stage('Helm dry-run') {
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

        stage('Deploy NGINX') {
            steps {
                container('helm') {
                    sh '''
                      helm upgrade --install $RELEASE $CHART \
                        --namespace $NAMESPACE \
                        --create-namespace \
                        --wait \
                        --timeout 10m
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ NGINX deployed successfully via Helm"
        }
        failure {
            echo "❌ Deployment failed — check Helm logs"
        }
    }
}

