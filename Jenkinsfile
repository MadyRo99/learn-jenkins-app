pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a9655efe-f8a9-4314-a38b-bb28a9843e14'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
/*         stage('build') {
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
        } */
        stage('Run Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        echo 'Test stage'
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            echo 'Reporting unit test results:'
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
                        echo 'E2E stage'
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            echo 'Reporting Playwright test results:'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage ('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo 'Deploying to production: Site ID: $NETLIFY_SITE_ID'
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                '''
                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        }
        stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                echo 'E2E stage'
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    echo 'Reporting Playwright test results:'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage ('Staging to Prod Approval') {
            steps {
                timeout(15) {
                    input cancel: 'No, abort.', message: 'Ready to deploy?', ok: 'Yes, I\'m ready to deploy.'
                }
            }
        }
        stage ('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo 'Deploying to production: Site ID: $NETLIFY_SITE_ID'
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://learnjenkinsapp.netlify.app'
            }
            steps {
                echo 'E2E stage'
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    echo 'Reporting Playwright test results:'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright PROD E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
