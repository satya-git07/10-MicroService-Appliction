pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/satya-git07/10-MicroService-Appliction.git'
        DOCKER_HUB_USER = 'satyadockerhub07'
        CREDENTIALS_ID = 'docker-credentials'
        GOOGLE_CREDENTIALS = credentials('gcp-sa')
        PROJECT_ID = 'lyrical-bus-452711-c5'
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
                stage('Build frontend') {
                    steps {
                        dir('10-MicroService-Appliction/src/frontend') {
                            sh 'npm install --only=production'
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
                            sh "docker build -t ${DOCKER_HUB_USER}/adservice:latest ."
                        }
                    }
                }
                stage('Build checkoutservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/checkoutservice') {
                            sh "docker build -t ${DOCKER_HUB_USER}/checkoutservice:latest ."
                        }
                    }
                }
                stage('Build productcatalogservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/productcatalogservice') {
                            sh "docker build -t ${DOCKER_HUB_USER}/productcatalogservice:latest ."
                        }
                    }
                }
                stage('Build shippingservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/shippingservice') {
                            sh "docker build -t ${DOCKER_HUB_USER}/shippingservice:latest ."
                        }
                    }
                }
                stage('Build frontend Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/frontend') {
                            sh "docker build -t ${DOCKER_HUB_USER}/frontend:latest ."
                        }
                    }
                }
                stage('Build paymentservice Image') {
                    steps {
                        dir('10-MicroService-Appliction/src/paymentservice') {
                            sh "docker build -t ${DOCKER_HUB_USER}/paymentservice:latest ."
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
                                sh "docker push ${DOCKER_HUB_USER}/adservice:latest"
                            }
                        }
                    }
                }
                stage('Push checkoutservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh "docker push ${DOCKER_HUB_USER}/checkoutservice:latest"
                            }
                        }
                    }
                }
                stage('Push productcatalogservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh "docker push ${DOCKER_HUB_USER}/productcatalogservice:latest"
                            }
                        }
                    }
                }
                stage('Push shippingservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh "docker push ${DOCKER_HUB_USER}/shippingservice:latest"
                            }
                        }
                    }
                }
                stage('Push frontend Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh "docker push ${DOCKER_HUB_USER}/frontend:latest"
                            }
                        }
                    }
                }
                stage('Push paymentservice Image') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                                sh "docker push ${DOCKER_HUB_USER}/paymentservice:latest"
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
