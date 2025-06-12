pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '89a8dde4-e4b4-4df7-ac42-496c77ad0572'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage ('Docker') {

            steps {
                sh 'docker build -t my-playwright .'
            }
        }
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
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    
                    steps {
                        sh '''
                        echo "Test Stage"
                        test -f build/index.html
                        set JEST_JUNIT_OUTPUT_DIR=jest-results
                        set JEST_JUNIT_OUTPUT_NAME=junit.xml
                        npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

         stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
             environment {
                        CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
                    }
            steps {                sh '''
                netlify --version
                echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                netlify status
                # Install node-jq globally to ensure it's available
                npm install -g node-jq
                netlify deploy --dir=build --json >deploy-output.json
                node-jq -r '.deploy_url' deploy-output.json
                '''
                script {
                    def deployOutput = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true).trim()
                    env.CI_ENVIRONMENT_URL = deployOutput
                    sh "npx playwright test --reporter=html"
                }
            }
           post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }

        }
        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Approve deployment to production?', ok: 'Deploy Now'
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
              environment {
                        CI_ENVIRONMENT_URL = 'https://beautiful-chaja-e4fbb0.netlify.app'
                    }
            steps {
                sh '''
                netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --prod 
                npx playwright test --reporter=html
                '''
            }
             post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    } // closes stages
} // closes pipeline