pipeline {
    agent any
    options {
        skipDefaultCheckout()
    }
    tools {
        maven 'maven-3.9.9' // Use the exact version form tool configuration
    }
    environment {
        //credentials for access between jenkins and gcp
        GOOGLE_CREDENTIALS = credentials('gcp-acess-for-jenkins')   // Credential ID for the Google Service Account
        GITHUB_CREDENTIALS = credentials('github-access-id') // Credential ID for GitHub token
        //GCP details
        GOOGLE_PROJECT_ID = 'multi-micro-project'
        GSA_EMAIL = 'jenkins-gsa@jenkins-gke-project-457719.iam.gserviceaccount.com'

        //cluster details
        GOOGLE_CLUSTER_NAME = 'dev-cluster'
        GOOGLE_CLUSTER_ZONE = 'us-central1-a'
        KSA_NAME = 'ksa'
        KSA_NAMESPACE = 'dn'

       //docker image details
        IMAGE_URL = 'asia-south1-docker.pkg.dev/multi-micro-project/mutli-micro-repo/order-service-project'

        DOCKER_BUILDKIT = '1'
        DOCKER_CLI_EXPERIMENTAL = 'enabled'
        BUILD_NUMBER_KEY = "${BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'github-access-id', url: 'https://github.com/Ravindra-87/order-service.git'
            }
        }

     stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

     /*
        stage('Build Docker Image') {
            steps {
                // Create and use a builder if not already
                sh 'docker buildx create --name jenkinsbuilders --use || echo "Builder already exists"'
                sh 'gcloud auth activate-service-account --key-file=$GOOGLE_CREDENTIALS'
                sh 'gcloud auth configure-docker asia-south1-docker.pkg.dev --quiet'

                echo "Full Image URL: ${IMAGE_URL}"

                // Build the image for amd64
                sh 'docker buildx build --platform linux/amd64 -t $IMAGE_URL:${BUILD_NUMBER} --push .'

            }
        }*/

        //  Deploy to GKE using kubectl
        stage('Deploy to GKE') {
            steps {
                script {
                    // Ensure kubectl is configured to use the correct GKE context
                    sh """
                       # Authenticate and set the kubeconfig for kubectl
                        gcloud container clusters get-credentials $GOOGLE_CLUSTER_NAME --zone $GOOGLE_CLUSTER_ZONE --project $GOOGLE_PROJECT_ID

                        kubectl config set-context --current --namespace=$KSA_NAMESPACE
                     
                        kubectl apply -f ./kuberenetes/kafka-server/kafka-deployment.yaml
                        kubectl apply -f ./kuberenetes/ingress/ingress-resource.yaml
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished. Check logs above for success or failure........'
        }
    }
}