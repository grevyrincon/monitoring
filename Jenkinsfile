pipeline {
  agent any

  environment {
    ENV = "prod"
    HELM_MONITORING_RELEASE = "monitoring"
    SECRET_NAME = 'aws-cred'
    AWS_REGION = 'us-east-1'
    TF_BUCKET = "terraform-state-bucket-eks-cluster"  
    TF_KEY = "${ENV}/terraform.tfstate"  
  }

  stages {
    
    stage('Load Environment from Terraform State') {
      steps {
        withAWS(region: "${AWS_REGION}", credentials: "${SECRET_NAME}") {
          script {
              // Download the tfstate file
              sh "aws s3 cp s3://${TF_BUCKET}/${TF_KEY} ./terraform.tfstate"

              // Read the state file JSON
              def tfStateText = readFile('terraform.tfstate')
              def tfState = readJSON text: tfStateText
              def outputs = tfState.outputs

              env.KUBE_CLUSTER = outputs.cluster_name.value
              env.AWS_REGION = outputs.aws_region.value

          }
        }
      }
    }
    stage('Create Grafana Dashboard ConfigMap') {
      steps {
        withAWS(region: "${AWS_REGION}", credentials: "${SECRET_NAME}") {
          sh """
            # Crear namespace si no existe
            kubectl get ns observability >/dev/null 2>&1 || kubectl create ns observability

            # Crear o actualizar el ConfigMap a partir del JSON
            kubectl create configmap container-dashboard \
              --from-file=dashboard/container-dashboard.json \
              -n observability \
              --dry-run=client -o yaml \
              | kubectl apply -f -

            # Añadir la etiqueta que el sidecar necesita
            kubectl label configmap container-dashboard grafana_dashboard=1 -n observability --overwrite
          """
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
