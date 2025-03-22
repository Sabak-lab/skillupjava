pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sabak-lab/skillupjava.git'
            }
        }
        stage('Build') {
            steps {
                echo "Building the application..."
            }
        }
    }
}
