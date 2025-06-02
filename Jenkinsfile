pipeline {
    agent any

    stages {
        stage('Build') {
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
        }
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
        if [ -f build/index.html ]; then
            echo "File exists."
        else
            echo "File does not exist."
            exit 1
        fi
        npm test
        '''
    }
}
    }
}