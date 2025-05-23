pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/sgandeplli/10-MicroService-Appliction.git'
        DOCKER_HUB_USER = 'satyadockerhub07'
        CREDENTIALS_ID = 'docker-credentials'  // Updated credentials ID
        GOOGLE_CREDENTIALS = credentials('gcp-sa')  
        // Docker image names for each microservice
        AD_SERVICE_IMAGE = 'satyadockerhub07/adservice:latest3'
        CHECKOUT_SERVICE_IMAGE = 'satyadockerhub07/checkoutservice:latest3'
        EMAIL_SERVICE_IMAGE = 'satyadockerhub07/emailservice:latest3'
        LOADGEN_SERVICE_IMAGE = 'satyadockerhub07/loadgenerator:latest3'
        PRODUCT_SERVICE_IMAGE = 'satyadockerhub07/productcatalogservice:latest3'
        SHIPPING_SERVICE_IMAGE = 'satyadockerhub07/shippingservice:latest3'
        CART_SERVICE_IMAGE = 'satyadockerhub07/cartservice:latest3'
        CURRENCY_SERVICE_IMAGE = 'satyadockerhub07/currencyservice:latest3'
        FRONTEND_SERVICE_IMAGE = 'satyadockerhub07/frontend:latest3'
        PAYMENT_SERVICE_IMAGE = 'satyadockerhub07/paymentservice:latest3'
        RECOMMENDATION_SERVICE_IMAGE = 'satyadockerhub07/recommendationservice:latest3'

        // Terraform Variables
        PROJECT_ID = 'lyrical-bus-452711-c5'  // Your GCP project ID
        REGION = 'us-west3-c'      // GCP region
        CLUSTER_NAME = 'my-cluster11' // GKE cluster name
        NODE_COUNT = 2                // Number of nodes in the GKE cluster
        NODE_MACHINE_TYPE = 'e2-standard-4'  // Machine type for the nodes
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
                    sh 'git clone --verbose --progress --recurse-submodules --depth=1 ${REPO_URL}'
                }
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
                stage('Build checkoutservice') {
                    steps {
                        dir('10-MicroService-Appliction/src/checkoutservice') {
                            sh 'go build -o checkoutservice .'
                        }
                    }
                }
                stage('Build productcatalogservice') {
                    steps {
                        dir('10-MicroService-Appliction/src/productcatalogservice') {
                            sh 'go build -o productcatalogservice .'
                        }
                    }
                }
                stage('Build shippingservice') {
                    steps {
                        dir('10-MicroService-Appliction/src/shippingservice') {
                            sh 'go build -o shippingservice .'
                        }
                    }
                }
                stage('Build cartservice') {
                    steps {
                        dir('10-MicroService-Appliction/src/cartservice/src') {
                            sh 'dotnet restore'
                            sh 'dotnet build -c Release'
                            sh 'dotnet publish -c Release -o out'
                        }
                    }
                }
                stage('Build currencyservice') {
                    steps {
                        dir('10-MicroService-Appliction/src/currencyservice') {
                             sh 'npm install --only=production'
                        }
                    }
                }
                stage('Build frontend') {
                    steps {
                        dir('10-MicroService-Appliction/src/frontend') {
                            sh 'go build -o frontend .'
                        }
                    }
                }
                stage('Build paymentservice') {
                    steps {
                        dir('10-MicroService-Appliction/src/paymentservice') {
                            sh 'npm install --only=production'
                        }
                    }
                }
                
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build adservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/adservice') {
                            sh 'docker build -t ${AD_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build checkoutservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/checkoutservice') {
                            sh 'docker build -t ${CHECKOUT_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build emailservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/emailservice') {
                            sh 'docker build -t ${EMAIL_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build loadgenerator Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/loadgenerator') {
                            sh 'docker build -t ${LOADGEN_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build productcatalogservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/productcatalogservice') {
                            sh 'docker build -t ${PRODUCT_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build shippingservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/shippingservice') {
                            sh 'docker build -t ${SHIPPING_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build cartservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/cartservice/src') {
                            sh 'docker build -t ${CART_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build currencyservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/currencyservice') {
                            sh 'docker build -t ${CURRENCY_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build frontend Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/frontend') {
                            sh 'docker build -t ${FRONTEND_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build paymentservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/paymentservice') {
                            sh 'docker build -t ${PAYMENT_SERVICE_IMAGE} .'
                        }
                    }
                }
                stage('Build recommendationservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/recommendationservice') {
                            sh 'docker build -t ${RECOMMENDATION_SERVICE_IMAGE} .'
                        }
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            parallel {
                stage('Push adservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${AD_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push checkoutservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${CHECKOUT_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push emailservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${EMAIL_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push loadgenerator Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${LOADGEN_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push productcatalogservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${PRODUCT_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push shippingservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${SHIPPING_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push cartservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${CART_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push currencyservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${CURRENCY_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push frontend Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${FRONTEND_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push paymentservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${PAYMENT_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
                stage('Push recommendationservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh 'docker push ${RECOMMENDATION_SERVICE_IMAGE}'
                            }
                        }
                    }
                }
            }
        }

      
        
                stage('Terraform: Apply Infrastructure') {
             steps {
                script {
                    echo 'Applying Terraform configurations to create GCP resources...'
                    // Ensure you're authenticated and have the necessary permissions to create resources
                    withCredentials([file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        // Authenticate with Google Cloud
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
        
                        // Set the environment variable for Terraform GCP provider to use
                        sh 'export GOOGLE_APPLICATION_CREDENTIALS=$GOOGLE_APPLICATION_CREDENTIALS'
        
                        // Change directory to where the Terraform configuration files are located
                        dir('10-MicroService-Appliction/terra') {
                            // Initialize Terraform
                            sh 'terraform init'
        
                            // Apply Terraform plan to create resources
                            sh 'terraform apply -auto-approve -var="project_id=${PROJECT_ID}" -var="region=${REGION}" -var="cluster_name=${CLUSTER_NAME}" -var="node_count=${NODE_COUNT}" -var="node_machine_type=${NODE_MACHINE_TYPE}"'
                        }
                    }
                }
            }
        }

        
        
            stage('Deploy to GKE') {
                steps {
                    script {
                        // Authenticate with Google Cloud
                        withCredentials([file(credentialsId: 'gcp-sa', variable: 'GCP_KEY')]) {
                            sh 'gcloud auth activate-service-account --key-file=$GCP_KEY'
                            
                            // Set the project and region
                            sh 'gcloud config set project ${PROJECT_ID}'
                            sh 'gcloud config set compute/region ${REGION}'
                            
                            // Get credentials for kubectl to access the GKE cluster
                            sh 'gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${REGION}'
                            
                            // Change directory to where the Kubernetes deployment manifest file is located
                            dir('10-MicroService-Appliction/k8s') {
                                // Deploy the application using kubectl
                                sh 'kubectl apply -f deploy.yaml'
                            }
                        }
                    }
                }
            }



        

    }
}
