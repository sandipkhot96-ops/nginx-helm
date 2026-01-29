pipeline {
    agent {
        kubernetes {
            label 'helm-nginx-agent'
            defaultContainer 'jnlp'
        }
    }

    environment {
        NAMESPACE = "nginx"
        RELEASE   = "nginx"
        CHART     = "nginx"
        REPO      = "https://charts.bitnami.com/bitnami"
    }

    stages {

        stage('Verify Tools') {
            steps {
                sh '''
                  kubectl version --client
                  helm version
                '''
            }
        }

        stage('Prepare Kubernetes Namespace') {
            steps {
                sh '''
                  kubectl get namespace ${NAMESPACE} \
                  || kubectl create namespace ${NAMESPACE}
                '''
            }
        }

        stage('Deploy NGINX using Helm') {
            steps {
                sh '''
                  helm repo add bitnami ${REPO}
                  helm repo update

                  helm upgrade --install ${RELEASE} bitnami/${CHART} \
                    --namespace ${NAMESPACE} \
                    --values values.yaml \
                    --wait \
                    --timeout 5m
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                  kubectl get pods -n ${NAMESPACE}
                  kubectl get svc -n ${NAMESPACE}
                '''
            }
        }
    }
}
