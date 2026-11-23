pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'GitHub repository cloned successfully!'
            }
        }

        stage('Build') {
            steps {
                echo 'Build stage completed!'
            }
        }

        stage('Test') {
            steps {
                echo 'Test stage completed!'
            }
        }
    }
}
