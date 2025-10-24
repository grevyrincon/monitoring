pipeline {
  agent any

  environment {
    ENV = "prod"
    HELM_MONITORING_RELEASE = "monitoring"
    SECRET_NAME = 'aws-cred'
    AWS_REGION = 'us-east-1'
    KUBE_CLUSTER = "api-cluster-${ENV}"
  }

  stages {
    
    stage('Validate AWS') {
      steps {
        withAWS(region: "${AWS_REGION}", credentials: "${SECRET_NAME}") {
          sh 'aws --version'
          sh 'aws sts get-caller-identity'
          sh 'aws eks list-clusters'
        }
      }
    }

    stage('Deploy Monitoring Stack') {
      steps {
        withAWS(region: "${AWS_REGION}", credentials: "${SECRET_NAME}") {
          sh """
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${KUBE_CLUSTER}

            helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
            helm repo update
            
            helm upgrade --install ${HELM_MONITORING_RELEASE} prometheus-community/kube-prometheus-stack \
              --namespace observability \
              --create-namespace \
              -f values-monitoring.yaml
          """
        }
      }
    }    
    
  }

  post {
    success {
      echo "Deploy successful"
    }
    failure {
      echo "Pipeline failed"
    }
  }
}
