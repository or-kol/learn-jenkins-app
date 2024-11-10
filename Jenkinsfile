pipeline {
    agent any

    environment {
        BUILD_FILE_NAME = 'index.html'
    }

    stages {
        stage('build') {
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

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                echo 'Testing jenkins app'
                sh '''
                    test -f build/$BUILD_FILE_NAME
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.48.1-noble'
                    reuseNode true
                }
            }

            steps {
                echo 'E2E testing jenkins app'
                sh '''
                    npm install -g serve
                    serve -s build
                    npx playwrite test
                '''
            }
        }
    }
    
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}

/*
npm install -g serve
serve -s build
*/