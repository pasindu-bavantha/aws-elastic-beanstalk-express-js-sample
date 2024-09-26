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
            }
        }

        stage('Snyk Security Install') {
            steps {
                sh 'npm install snyk --save-dev'
                echo 'Snyk Installed'
            }
        }

        stage('Snyk Authentication') {
            steps {
                withCredentials([string(credentialsId: 'snyk_token', variable: 'SNYK_TOKEN')]) {
                    sh './node_modules/.bin/snyk auth $SNYK_TOKEN'
                    echo 'Snyk Authentication Finished'
                }
            }
        }

        stage('Snyk Security Scan') {
            steps {
                 script {
                    def snykResults = sh(script: './node_modules/.bin/snyk test --json', returnStdout: true)
                    def jsonResults = readJSON(text: snykResults)
                    if (jsonResults.vulnerabilities.any { it.severity == 'critical' }) {
                        error("Critical vulnerabilities found! Check snyk-report.json.")
                    } else {
                        writeFile file: 'snyk-report.json', text: snykResults
                    }
                }

                echo 'Snyk Security Scan Completed'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'npm install'
                    echo 'Build completed.'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'npm test'
                    echo 'Tests completed.'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
