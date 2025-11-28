pipeline {
    agent any

    environment {
        PROJECT_ID = "tidy-landing-476507-h6"
        REGION = "us-west1"
        CLUSTER_NAME = "devops-gke"
        REPO_NAME = "petclinic-repo"
        IMAGE_NAME = "petclinic"
        IMAGE_TAG = "v1"
    }

    stages {
        
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/testigithubrit123/spring-petclinic.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        echo "Activating GCP Service Account..."
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS

                        echo "Setting GCP project..."
                        gcloud config set project $PROJECT_ID

                        echo "Getting GKE Credentials..."
                        gcloud container clusters get-credentials $CLUSTER_NAME --region $REGION
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker Image..."
                    docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Image to Artifact Registry') {
            steps {
                sh '''
                    echo "Configuring Docker authentication for Artifact Registry..."
                    gcloud auth configure-docker $REGION-docker.pkg.dev -q
                    
                    echo "Pushing Docker Image..."
                    docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh '''
                    echo "Updating deployment.yaml with new image..."

                    sed -i "s#IMAGE_HERE#$REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG#g" k8s/deployment.yaml

                    echo "Applying Kubernetes manifests..."
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}

       
