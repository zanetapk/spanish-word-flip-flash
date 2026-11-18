pipeline {
    agent any
    
    options {
        ansiColor('xterm')
    }

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:22-alpine'
                }
            }
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('test') {
            parallel {
                stage('unit tests') {
                    agent {
                        docker {
                            image 'node:22-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        // Unit tests with Vitest
                        sh 'npm ci'
                        sh 'npx vitest run --reporter=verbose'
                    }
                }
                stage('integration tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        // Integration tests with Playwright
                        sh 'npm ci'
                        sh 'npx playwright test'
                    }
                }
            }
        }

        stage('deploy') {
            agent {
                docker {
                    image 'alpine'
                }
            }
            steps {
                // Mock deployment which does nothing
                echo 'Mock deployment was successful!'
            }
        }

        stage('e2e') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                    reuseNode true
                }
            }
            environment {
                // Set environment variables for Playwright tests
                E2E_BASE_URL = 'https://spanish-cards.netlify.app/'
            }

            steps {
                sh 'npm ci'
                sh 'npx playwright test'
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        icon: '',
                        keepAll: false,
                        reportDir: 'reports-e2e/html/',
                        reportFiles: 'index.html',
                        reportName: 'Playwright HTML Report',
                        reportTitles: '',
                        useWrapperFileDirectory: true
                    ])
                    junit studioRetention: 'ALL', testResults: 'reports-e2e/junit.xml'
                }
            }
        }
    }
}