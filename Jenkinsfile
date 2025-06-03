pipeline {
    agent any

    stages {
        /* stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine' // Use Node.js 18 Alpine image
                    reuseNode true // Reuse the same node for this stage
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
        } */
        stage('Test') {
             agent {
                docker {
                    image 'node:18-alpine' // Use Node.js 18 Alpine image
                    reuseNode true // Reuse the same node for this stage
                }
            }
            steps {
                    sh '''
                    echo "Test Stage"
                    test -f build/index.html
                    npm test
                    '''
    }
    stage('E2E') {
             agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-noble' // Use Node.js 18 Alpine image
                    reuseNode true // Reuse the same node for this stage
                }
            }
            steps {
                    sh '''
                    npm install -g serve
                    serve -s build &
                    npx playwright test
                    '''
    }
}
    
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}