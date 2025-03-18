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
