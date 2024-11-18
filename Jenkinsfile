pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = "bc6d044f-eadb-4bfc-afbb-1bea5221af78"
        NETLIFY_AUTH_TOKEN = credentials("netlify-token")
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage("Docker Build") {
            steps {
                sh "docker build -t app ."
            }
        }

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
                            image "app"
                            reuseNode true
                        }
                    }

                    steps {
                        sh'''
                            serve -s build &
                            sleep 15
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage("Deploy Staging") {
            agent {
                docker {
                    image "app"
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "UNDEFINED"
            }

            steps {
                sh'''
                    netlify --version
                    echo "Deploying to Staging: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-stag.json
                    CI_ENVIRONMENT_URL=$(node-jq -r ".deploy_url" deploy-stag.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Stag HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage("Deploy Prod") {
            agent {
                docker {
                    image "app"
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "https://brilliant-kataifi-94030b.netlify.app"
            }

            steps {
                sh'''
                    node --version
                    netlify --version
                    echo "Deploying to Prod: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Prod HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}