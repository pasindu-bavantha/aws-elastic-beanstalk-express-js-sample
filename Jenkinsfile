pipeline {
    agent {
        docker {
            image 'node:16'  // Use Node.js Docker image as the build agent
        }
    }
    stages {
        stage('Install Dependencies') {
            steps {
                // Install project dependencies
                sh 'npm install'
            }
        }
        stage('Build') {
            steps {
                // Run the build command
                sh 'npm run build'
            }
        }
        stage('Test') {
            steps {
                // Run the tests
                sh 'npm test'
            }
        }
        stage('Security Scan') {
            steps {
                // Install Snyk globally
                sh 'npm install -g snyk'
                
                // Authenticate with Snyk (Use API token or credentials in Jenkins for real usage)
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh 'snyk auth $SNYK_TOKEN'
                }
                
                // Run Snyk security scan
                sh 'snyk test --severity-threshold=high'
            }
        }
        stage('Deploy') {
            steps {
                // Simulate deployment
                echo 'Deployment stage...'
            }
        }
    }
    post {
        failure {
            // If any stage fails, stop the pipeline
            echo 'Pipeline failed, check the logs for details!'
        }
        always {
            // Always executed after pipeline
            echo 'Pipeline complete!'
        }
    }
}

          
