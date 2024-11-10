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

            '''
            test if the following exist:
            build/index.html file
            npm test
            a
            '''
        stage('Test') {
            steps {
                echo 'Testing jenkins app'
                sh '''
                    test -f build/$BUILD_FILE_NAME
                    npm test
                    a
                '''
            }
        }
    }
}
