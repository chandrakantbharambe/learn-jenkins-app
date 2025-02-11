pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '46bedf02-5d5d-4a49-b784-9278f6922758'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        MY_VAR = 'Start_Value'
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
                    echo 'Small Change'
                    ls -la
                    npm --version 
                    node --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit Tests') {
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
                            sleep 10s
                            npx playwright test
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        // stage('Deploy Staging') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //            npm install netlify-cli node-jq
        //            node_modules/.bin/netlify --version
        //            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //            node_modules/.bin/netlify status
        //            node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
        //            echo "Staging"
        //         '''
        //         script {
        //             env.MY_VAR = sh(script: 'date', returnStdout: true)
        //             env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
        //         }
        //     }
        // }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                 CI_ENVIRONMENT_URL="START_VALUE_TO_CHECK_IS_IT_GOING_UPDATE_IN_SH"
            }

            steps {

                 sh '''
                   npm install netlify-cli node-jq
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                   CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                   npx playwright test --reporter=html
                   MY_VAR=$(date)
                   echo "Staging"
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        // stage('Approval') {
        //     steps {
        //         timeout(time:15, unit: 'MINUTES') {
        //             input message: 'Pipeline is ready for deploy on production?', ok: 'Yes! It is ready'
        //         }
        //     }
        // }

        // stage('Deploy Prod') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //          steps {
        //         sh '''
        //            npm install netlify-cli
        //            node_modules/.bin/netlify --version
        //            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //            node_modules/.bin/netlify status
        //            node_modules/.bin/netlify deploy --dir=build --prod
        //            echo "Prod"
        //         '''
        //          echo "MY_VAR is: $env.MY_VAR"
        //     }
        //     }
        // }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                 CI_ENVIRONMENT_URL='https://learn-jenkin.netlify.app'
            }

            steps {
                sh '''
                   npm install netlify-cli
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build --prod
                   npx playwright test --reporter=html
                   echo "Prod"
                '''
                 echo "MY_VAR is : $env.MY_VAR"
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }
}