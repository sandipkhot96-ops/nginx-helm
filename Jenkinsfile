pipeline {
  agent {
    kubernetes {
      label 'helm-nginx'
      defaultContainer 'jnlp'
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

    stage('Helm lint (safe check)') {
      steps {
        container('helm') {
          sh '''
            helm lint .
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
              -f values.yaml \
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
              -f values.yaml \
              --wait \
              --timeout 5m
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ NGINX deployed safely using Helm"
    }
    failure {
      echo "❌ Deployment failed — no destructive action taken"
    }
  }
}
