pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = 'bc6d044f-eadb-4bfc-afbb-1bea5221af78'
    }

    stages {
        stage("Build") {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            
            steps {
                sh'''
                    echo "Docker Container Started!"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage("Tests") {
            parallel {
                stage("Unit Test") {
                    agent {
                        docker {
                            image "node:18-alpine"
                            reuseNode true
                        }
                    }

                    steps {
                        sh'''
                            echo "Test stage"
                            test -f build/index.html
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit "jest-results/junit.xml"
                        }
                    }
                }

                stage("E2E Test") {
                    agent {
                        docker {
                            image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                            reuseNode true
                        }
                    }

                    steps {
                        sh'''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playWright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage("Deploy") {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }

            steps {
                sh'''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying: $NETLIFY_SITE_ID"
                '''
            }
        }
    }
}