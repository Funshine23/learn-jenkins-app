pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '350365f6-b4de-4b43-a4d6-b96e183870ac'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
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
        
        stage('Tests') {
            parallel {
                //Test Build
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps { 
                        sh '''
                            echo 'Start Unit Testing'
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit testResults: 'jest-results/junit.xml', skipPublishingChecks: true                            
                        }
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
                            echo 'Start E2E Testing'
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright install chromium
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {                            
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }   

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli node-jq
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify status                
                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                '''
            }
        }

        stage('Approval') {            
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input 'Ready to deploy?'
                }                
            }
        }        

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install netlify-cli      
                node_modules/.bin/netlify --version
                node_modules/.bin/netlify status
                echo 'Deploying to production'
                node_modules/.bin/netlify deploy --dir=build --prod
                
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                    reuseNode true
                    //args '-u root:root'
                }                
            }

            environment {        
                CI_ENVIRONMENT_URL = 'https://transcendent-pixie-ae9eac.netlify.app'
            }

            steps { 
                sh '''                    
                    npx playwright install chromium
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {                            
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
