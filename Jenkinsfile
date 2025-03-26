pipeline {
    agent any

    environment {
        IMAGE_NAME = "skillup-java-app"
        CONTAINER_REGISTRY = "your-docker-hub-username/skillup-java"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Sabak-lab/skillupjava.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    echo "Building Docker Image..."
                    docker build -t ${IMAGE_NAME} .
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    script {
                        sh '''
                        echo "Tagging and Pushing Docker Image..."
                        docker tag ${IMAGE_NAME} ${CONTAINER_REGISTRY}:latest
                        docker push ${CONTAINER_REGISTRY}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    echo "Deploying to Kubernetes..."
                    cat <<EOF | kubectl apply -f -
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: skillup-java
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: skillup-java
                      template:
                        metadata:
                          labels:
                            app: skillup-java
                        spec:
                          containers:
                            - name: skillup-java
                              image: ${CONTAINER_REGISTRY}:latest
                              ports:
                                - containerPort: 8080
                    ---
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: skillup-java-service
                    spec:
                      selector:
                        app: skillup-java
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 8080
                      type: LoadBalancer
                    EOF
                    '''
                }
            }
        }

        stage('Expose Public URL') {
            steps {
                script {
                    sh "kubectl get services skillup-java-service -o=jsonpath='{.status.loadBalancer.ingress[0].hostname}'"
                }
            }
        }
    }
}
