pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
                echo 'Install Dependencies Finished'

                sh 'npm install snyk --save-dev'
                echo 'Snyk Installed'

                withCredentials([string(credentialsId: 'snyk_token', variable: 'SNYK_TOKEN')]) {
                    sh './node_modules/.bin/snyk auth $SNYK_TOKEN'
                    echo 'Snyk Authentication Finished'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm install --save'
                echo 'Installing Dependencies Finished'
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    def snykResults = sh(script: './node_modules/.bin/snyk test --json', returnStdout: true)
                    def jsonResults = readJSON(text: snykResults)
                    if (jsonResults.vulnerabilities.any { it.severity == 'critical' }) {
                        echo "Snyk Vulnerabilities found! Check snyk-report.json."
                        // Optionally write the results even if there are vulnerabilities
                        writeFile file: 'snyk-report.json', text: snykResults
                    } else {
                        writeFile file: 'snyk-report.json', text: snykResults
                    }
                }
        
                echo 'Snyk Scan Completed'
            }
            post {
                success {
                    echo 'Snyk Security Scan passed!'
                }
                failure {
                    echo 'Snyk Failed.'
                }
            }
        }
        }


        stage('Test') {
            steps {
                script {
                    sh 'npm install supertest'
                    sh 'npm test'
                    echo 'Test completed.'
                }
            }
            post {
                success {
                    echo 'Tests passed!'
                }
                failure {
                    echo 'Failed. Check logs for details.'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline Completed.'
        }
    }
}
