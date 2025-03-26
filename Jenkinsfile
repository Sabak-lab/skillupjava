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
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    script {
                        sh '''
                        echo "Logging in to Docker Hub..."
                        docker login -u your-docker-hub-username -p your-docker-hub-password

                        echo "Tagging and Pushing Docker Image..."
                        docker tag ${IMAGE_NAME}:latest ${CONTAINER_REGISTRY}:latest
                        docker push ${CONTAINER_REGISTRY}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'deployment.yaml', text: '''
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
                              image: your-docker-hub-username/skillup-java:latest
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
                    '''
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }

        stage('Expose Public URL') {
            steps {
                script {
                    sh '''
                    echo "Waiting for LoadBalancer IP..."
                    for i in {1..10}; do
                        IP=$(kubectl get services skillup-java-service -o=jsonpath='{.status.loadBalancer.ingress[0].hostname}')
                        if [ -n "$IP" ]; then
                            echo "Service is accessible at: http://$IP"
                            break
                        fi
                        echo "Retrying in 10 seconds..."
                        sleep 10
                    done
                    '''
                }
            }
        }
    }
}
