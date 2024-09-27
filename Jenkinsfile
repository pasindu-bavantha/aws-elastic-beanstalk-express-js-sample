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
                    // Run Snyk test with --ignore-policy and capture exit code
                    def snykResults = sh(script: './node_modules/.bin/snyk test --json --ignore-policy', returnStdout: true, returnStatus: true)
                    def jsonResults = readJSON(text: snykResults)
                    
                    // Check for vulnerabilities
                    if (jsonResults.vulnerabilities.any { it.severity == 'critical' }) {
                        echo "Snyk found critical vulnerabilities! Check snyk-report.json."
                        writeFile file: 'snyk-report.json', text: snykResults
                        // Optionally log detailed info or handle the findings
                    } else {
                        echo "No critical vulnerabilities found."
                        writeFile file: 'snyk-report.json', text: snykResults
                    }
                }
                echo 'Snyk Scan Completed'
            }
            post {
                success {
                    echo 'Snyk Security Scan completed!'
                }
                failure {
                    echo 'Snyk scan completed, but vulnerabilities were found.'
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
