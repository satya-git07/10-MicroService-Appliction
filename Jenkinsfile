pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/sgandeplli/10-MicroService-Appliction.git'
        DOCKER_HUB_USER = 'sekhar1913'
        CREDENTIALS_ID = 'docker-pass'
        GOOGLE_CREDENTIALS = credentials('gcp-key')
        PROJECT_ID = 'sekhar-452813'
        REGION = 'us-west3-c'
        CLUSTER_NAME = 'my-cluster11'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    sh 'git clone --depth=1 ${REPO_URL}'
                }
            }
        }

        stage('Build Microservices') {
            parallel {
                stage('Build adservice') {
                    steps {
                        dir('10-MicroService-Appliction/src/adservice') {
                            sh 'export JAVA_HOME=/usr/lib/jvm/java-21-openjdk && export PATH=$JAVA_HOME/bin:$PATH && ./gradlew build -x verifyGoogleJavaFormat'
                        }
                    }
                }
                stage('Build checkoutservice & others') {
                    matrix {
                        axes {
                            axis {
                                name 'SERVICE'
                                values 'checkoutservice', 'productcatalogservice', 'shippingservice', 'frontend', 'paymentservice'
                            }
                        }
                        stages {
                            stage('Build Service') {
                                steps {
                                    dir("10-MicroService-Appliction/src/${SERVICE}") {
                                        script {
                                            if (SERVICE == 'checkoutservice' || SERVICE == 'productcatalogservice' || SERVICE == 'shippingservice') {
                                                sh 'go build -o ${SERVICE} .'
                                            } else if (SERVICE == 'frontend' || SERVICE == 'paymentservice') {
                                                sh 'npm install --only=production'
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Docker Images for Services') {
                    matrix {
                        axes {
                            axis {
                                name 'SERVICE'
                                values 'adservice', 'checkoutservice', 'emailservice', 'loadgenerator', 'productcatalogservice', 'shippingservice', 'cartservice', 'currencyservice', 'frontend', 'paymentservice', 'recommendationservice'
                            }
                        }
                        stages {
                            stage('Build Docker Image') {
                                steps {
                                    dir("10-MicroService-Appliction/src/${SERVICE}") {
                                        sh "docker build -t ${DOCKER_HUB_USER}/${SERVICE}:latest ."
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            parallel {
                stage('Push Docker Images') {
                    matrix {
                        axes {
                            axis {
                                name 'SERVICE'
                                values 'adservice', 'checkoutservice', 'emailservice', 'loadgenerator', 'productcatalogservice', 'shippingservice', 'cartservice', 'currencyservice', 'frontend', 'paymentservice', 'recommendationservice'
                            }
                        }
                        stages {
                            stage('Push Image') {
                                steps {
                                    script {
                                        withCredentials([usernamePassword(credentialsId: 'docker-pass', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                            sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                            sh "docker push ${DOCKER_HUB_USER}/${SERVICE}:latest"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Terraform: Apply Infrastructure') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                        sh 'gcloud config set project ${PROJECT_ID}'
                        sh 'gcloud config set compute/region ${REGION}'

                        dir('10-MicroService-Appliction/terra') {
                            sh 'terraform init'
                            sh 'terraform apply -auto-approve -var="project_id=${PROJECT_ID}" -var="region=${REGION}" -var="cluster_name=${CLUSTER_NAME}"'
                        }
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                        sh 'gcloud config set project ${PROJECT_ID}'
                        sh 'gcloud config set compute/region ${REGION}'

                        sh 'gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${REGION}'

                        dir('10-MicroService-Appliction/k8s') {
                            sh 'kubectl apply -f deploy.yaml'
                        }
                    }
                }
            }
        }
    }
}
pipeline {
  agent any
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

  stages {
    stage('Git Checkout') {
      steps {
        git(url: 'https://github.com/satya-git07/10-MicroService-Appliction.git', branch: "${GIT_BRANCH}")
      }
    }

  stage('Build Microservices') {
            parallel {
                stage('Build adservice') {
                    steps {
                        dir('10-MicroService-Appliction/src/adservice') {
                            script {
                                sh 'export JAVA_HOME=/usr/lib/jvm/java-21-openjdk'   // Switch to Java 21
                                sh 'export PATH=$JAVA_HOME/bin:$PATH'
                                sh 'java -version'   // Verify the Java version is 21
                                sh 'chmod +x gradlew'
                                sh './gradlew build -x verifyGoogleJavaFormat'
                            }
                        }
                    }
                } 
            } 
  }

    stage('SonarQube Analysis') {
      steps {
        script {
          // Run SonarQube analysis using sonar-scanner
          sh """
            /opt/sonar-scanner-new/sonar-scanner-4.8.0.2856-linux/bin/sonar-scanner \
                -Dsonar.projectKey="microservices" \
                -Dsonar.sources="." \
                -Dsonar.host.url="http://192.168.2.179:9000" \
                -Dsonar.login="squ_ed34cb324eb747588d83f3543d713e9378421d68" \
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
}
