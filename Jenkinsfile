pipeline {
  agent any
  stages {
    stage('Git Checkout') {
      steps {
        git(url: 'https://github.com/satya-git07/10-MicroService-Appliction.git', branch: "${GIT_BRANCH}")
      }
    }

    
  stage('SonarQube Analysis') {
              steps {
                  script {
                      // Run SonarQube analysis using sonar-scanner
                      sh """
                          /opt/sonar-scanner-new/sonar-scanner-4.8.0.2856-linux/bin/sonar-scanner \
                              -Dsonar.projectKey="tiktoktoe" \
                              -Dsonar.sources="." \
                              -Dsonar.host.url="http://192.168.2.179:9000" \
                              -Dsonar.login="squ_ed34cb324eb747588d83f3543d713e9378421d68"
                              -Dsonar.java.binaries=target/classes
                      """
                  }
              }
          }

    
    stage('Docker Build & Push') {
      steps {
        script {
          MICRO_SERVICES.each { service ->
          echo "Building Docker image for ${service}"

          // Log in to Docker Hub
          sh """
          docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}
          """

          // Build, tag, and push the Docker image to Docker Hub
          sh """
          cd ${service} && docker build -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}/${service}:latest .
          docker tag ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}/${service}:latest ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}/${service}:latest
          docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}/${service}:latest
          """
        }
      }

    }
  }

  stage('Trivy Vulnerability Scan') {
    steps {
      script {
        MICRO_SERVICES.each { service ->
        echo "Scanning Docker image for ${service} using Trivy"
        sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}/${service}:latest > trivy_${service}_scan.txt"
      }
    }

  }
}

stage('Terraform Init') {
  steps {
    sh 'terraform init'
    sh 'ls -la'
  }
}

stage('Terraform Apply') {
  steps {
    withCredentials(bindings: [file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
      sh 'terraform apply -auto-approve'
    }

  }
}

stage('Deploy to GKE') {
  steps {
    withCredentials(bindings: [file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
      script {
        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
        sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${ZONE} --project ${PROJECT_ID}"

        // Deploy each microservice using kubectl and the corresponding YAML file
        MICRO_SERVICES.each { service ->
        echo "Deploying ${service} to GKE"
        sh "kubectl apply -f kubernetes-manifests/${service}.yaml"
      }

      // Optional: Apply the kustomization.yaml if needed
      sh "kubectl apply -k kubernetes-manifests/"
    }

  }

}
}

}
environment {
CLUSTER_NAME = 'my-cluster1'
ZONE = 'us-west3'
PROJECT_ID = 'lyrical-bus-452711-c5'
GIT_BRANCH = 'main'
GOOGLE_CREDENTIALS_ID = 'gcp-sa'
DOCKERHUB_USERNAME = 'satyadockerhub07'
DOCKERHUB_REPO = 'satyadockerhub07/microservices'
AD_SERVICE_IMAGE = 'adservice'
CHECKOUT_SERVICE_IMAGE = 'checkoutservice'
EMAIL_SERVICE_IMAGE = 'emailservice'
LOADGEN_SERVICE_IMAGE = 'loadgenerator'
PRODUCT_SERVICE_IMAGE = 'productcatalogservice'
SHIPPING_SERVICE_IMAGE = 'shippingservice'
CART_SERVICE_IMAGE = 'cartservice'
CURRENCY_SERVICE_IMAGE = 'currencyservice'
FRONTEND_SERVICE_IMAGE = 'frontend'
PAYMENT_SERVICE_IMAGE = 'paymentservice'
RECOMMENDATION_SERVICE_IMAGE = 'recommendationservice'
}
}
