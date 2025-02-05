pipeline {
    agent any

    stages {
        // Stage Build
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la
                node --version
                npm --version
                npm ci 
                npm run build
                ls -la
                '''
            }
        }
        //Test Build
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps { 
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }
        //E2E Build
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                    reuseNode true
                    //args '-u root:root'
                }
            }
            steps { 
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 60
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            junit testResults: 'jest-results/junit.xml', skipPublishingChecks: true
        }
    }
}
