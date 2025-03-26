pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sabareesh954/skillupjava' 
        KUBE_DEPLOYMENT = 'skillupjava-deployment'
        KUBE_SERVICE = 'skillupjava-service'
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
                    bat 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/']) {
                        bat 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl apply -f - <<EOF
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${KUBE_DEPLOYMENT}
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: skillupjava
                      template:
                        metadata:
                          labels:
                            app: skillupjava
                        spec:
                          containers:
                          - name: skillupjava
                            image: ${DOCKER_IMAGE}
                            ports:
                            - containerPort: 8080
                    ---
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: ${KUBE_SERVICE}
                    spec:
                      type: LoadBalancer
                      selector:
                        app: skillupjava
                      ports:
                        - protocol: TCP
                          port: 80
                          targetPort: 8080
                    EOF
                    """
                }
            }
        }

        stage('Expose Public URL') {
            steps {
                script {
                    sh 'kubectl get services ${KUBE_SERVICE} -o jsonpath="{.status.loadBalancer.ingress[0].hostname}"'
                }
            }
        }
    }
}
