pipeline {
    agent any

    environment {
        IMAGE_NAME = "skillup-java-app"
        CONTAINER_REGISTRY = "your-docker-hub-username/skillup-java"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git credentialsId: '79482a8e-12af-4ec7-9f47-c82f4db812f7', url: 'https://github.com/Sabak-lab/skillupjava.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${env.CONTAINER_REGISTRY} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    bat "docker push ${env.CONTAINER_REGISTRY}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
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
                          image: ${env.CONTAINER_REGISTRY}
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
                bat "minikube service skillup-java-service --url"
            }
        }
    }
}
