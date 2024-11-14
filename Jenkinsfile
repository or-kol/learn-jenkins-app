pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'fe7cf59e-29b6-402e-9811-fa9da4fabee0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
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

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.15.53'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }

            environment {
                AWS_S3_BUCKET_NAME = 'learn-jenkins-202411140958'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws s3 sync build s3://$AWS_S3_BUCKET_NAME
                    '''
                }
            }
        }

        stage('Tests'){
            parallel {
                stage('Unit test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'Junit-test-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'my-plawright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=line
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', 
                                reportName: 'Playwright local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-plawright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }
            
            steps {
                sh '''
                    netlify --version
                    echo "Deploy to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
                }
            }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-plawright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://dainty-druid-e88c9c.netlify.app'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploy to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    sleep 10
                    npx playwright test --reporter=line
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', 
                        reportName: 'Playwright prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }    
}
