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
                sh '''
                echo "Building Docker Image..."
                docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh "docker tag ${IMAGE_NAME} ${CONTAINER_REGISTRY}"
                    sh "docker push ${CONTAINER_REGISTRY}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl apply -f - <<EOF
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
                          image: ${CONTAINER_REGISTRY}
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
                """
            }
        }

        stage('Expose Public URL') {
            steps {
                sh "minikube service skillup-java-service --url"
            }
        }
    }
}



