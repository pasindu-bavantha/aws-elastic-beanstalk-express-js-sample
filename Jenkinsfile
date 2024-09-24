pipeline {
    agent {
        docker {
            image 'node:16'
            args '-u root:root'
        }
    }
    stages {
        stage('Install dependencies') {
            steps {
                sh 'npm install --save'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
