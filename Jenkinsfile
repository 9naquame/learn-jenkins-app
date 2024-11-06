pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = "bc6d044f-eadb-4bfc-afbb-1bea5221af78"
        NETLIFY_AUTH_TOKEN = credentials("netlify-token")
        CI_ENVIRONMENT_URL = "https://brilliant-kataifi-94030b.netlify.app/"
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
                    image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "UNDEFINED"
            }

            steps {
                sh'''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to Staging: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/netlify deploy --dir=build --json > deploy-stag.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Stag HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage("Manual Approval") {
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: "Do you wish to deploy to roduction?", ok: "Yes, I am sure!"
                }
            }
        }

        stage("Deploy Prod") {
            agent {
                docker {
                    image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "https://brilliant-kataifi-94030b.netlify.app"
            }

            steps {
                sh'''
                    node --version
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to Prod: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
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