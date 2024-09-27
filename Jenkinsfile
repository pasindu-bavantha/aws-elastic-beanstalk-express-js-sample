pipeline {
    agent {
        docker {
            image 'node:16'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
                echo 'Dependency installed'

                sh 'npm install snyk --save-dev'
                echo 'Snyk installation completed'

            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
                echo 'Build completed'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                script {
                    def snykPoint = sh(script: './node_modules/.bin/snyk test --json', returnStdout: true)
                    def jsonPoint = readJSON(text: snykPoint)
                    if (jsonPoint.vulnerabilities.any { it.severity == 'critical' }) {
                        error("Critical vulnerabilities found!")
                    } else {
                        writeFile file: 'snyk-report.json', text: snykPoint
                    }
                }

                echo 'Security scan completed'
            }
            post {
                success {
                    echo 'Security scan passed!'
                }
                failure {
                    echo 'Security scan failed due to critical vulnerabilities.'
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
            post {
                success {
                    echo 'Tests passed!'
                }
                failure {
                    echo 'Some tests failed. Check logs for details.'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
            archiveArtifacts artifacts: '**/snyk-report.json', allowEmptyArchive: true
        }
    }
}
