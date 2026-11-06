pipeline {
    agent any
    tools {
        jdk 'jdk21'
        nodejs 'node23'
    }
environment {
       // SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'mahalaxmi08/zomato:latest'
        EKS_CLUSTER_NAME = 'zomato-cluster'
        AWS_REGION = 'ap-southeast-2'
    }

    stages {
        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git 'https://github.com/mahalaxmi1308/Zomato.git'
            }
        }
       
        
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
      
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t zomato ."
            }
        }
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker tag zomato mahalaxmi08/zomato:latest "
                        sh "docker push mahalaxmi08/zomato:latest "
                    }
                }
            }
       
        stage ("Deploy to Container") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 mahalaxmi08/zomato:latest'
            }
        }
       stage('Deploy to EKS Cluster') {
            steps {
                script {
                    sh '''
                    echo "Verifying AWS credentials..."
                    aws sts get-caller-identity

                    echo "Configuring kubectl for EKS cluster..."
                    aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION

                    echo "Verifying kubeconfig..."
                    kubectl config view

                    echo "Deploying application to EKS..."
                    kubectl apply -f Kubernetes/deployment.yaml
                    kubectl apply -f Kubernetes/service.yaml
                    kubectl apply -f Kubernetes/node-service.yaml

                    echo "Verifying deployment..."
                    kubectl get pods
                    kubectl get svc
                    '''
                }
            }
        }
    
