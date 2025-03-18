pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = 'f7a94671-82dc-42c2-9f67-c92f3915b0d0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        
    }
    stages {
        stage('Docker'){
            steps{
                sh 'docker build -t my-playwright .'
            }
        }
        stage('Build') {
            agent{
                docker{
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Run tests'){
            parallel {
                stage('Unit Tests'){
                    agent{
                        docker{
                            image "node:18-alpine"
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            echo "Test stage"
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E'){
                    agent{
                        docker{
                            image "my-playwright"
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging'){
            agent{
                docker{
                    image "my-playwright"
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
            }
            steps{
                sh '''
                    netlify --version
                    echo "Deplying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage("Approval") {
            steps{
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }
        stage('Deploy Prod'){
            agent{
                docker{
                    image "my-playwright"
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://steady-gelato-746b04.netlify.app'
            }
            steps{
                sh '''
                    node --version
                    netlify --version
                    echo "Deplying to prod. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
    
}
